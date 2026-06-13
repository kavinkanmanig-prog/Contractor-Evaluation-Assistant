import streamlit as st
from io import BytesIO
from pypdf import PdfReader
from langchain_community.chat_models import ChatOllama

# -------------------------
# CONFIG
# -------------------------
st.set_page_config(page_title="CMRL AI", layout="wide")
st.title("🤖 CMRL Contractor AI Assistant")

# -------------------------
# SESSION STATE
# -------------------------
if "messages" not in st.session_state:
    st.session_state.messages = []

if "data" not in st.session_state:
    st.session_state.data = None

# -------------------------
# OLLAMA MODEL
# -------------------------
llm = ChatOllama(model="gemma3")

# -------------------------
# FILE UPLOAD
# -------------------------
uploaded_files = st.file_uploader(
    "Upload Contractor PDFs",
    type=["pdf"],
    accept_multiple_files=True
)

# -------------------------
# SAFE DATA EXTRACTION
# -------------------------
def extract_data(text):
    data = {
        "name": "Unknown",
        "cost": 0,
        "experience": 0,
        "safety": 0,
        "projects": 0
    }

    lines = text.split("\n")

    for line in lines:
        if ":" not in line:
            continue

        parts = line.split(":")
        if len(parts) < 2:
            continue

        key = parts[0].strip()
        value = parts[1].strip()

        try:
            if "Company Name" in key:
                data["name"] = value

            elif "Cost" in key:
                data["cost"] = int(value)

            elif "Experience" in key:
                data["experience"] = int(value)

            elif "Safety Rating" in key:
                data["safety"] = int(value)

            elif "Projects Completed" in key:
                data["projects"] = int(value)

        except:
            continue

    return data

# -------------------------
# LOAD DATA (ONCE)
# -------------------------
def load_data(files):
    contractors = []

    for file in files:
        reader = PdfReader(BytesIO(file.read()))
        text = ""

        for page in reader.pages:
            text += page.extract_text()

        contractors.append(extract_data(text))

    return contractors

# Load only once
if uploaded_files and st.session_state.data is None:
    st.session_state.data = load_data(uploaded_files)

# -------------------------
# SCORING
# -------------------------
def calculate_score(d):
    score = 0

    if d["experience"] > 5:
        score += 30

    if d["cost"] < 50:
        score += 30

    score += d["safety"] * 5
    score += min(d["projects"] * 2, 20)

    return score

# -------------------------
# AI QUERY UNDERSTANDING
# -------------------------
def interpret_query(query):
    prompt = f"""
    Convert this user request into filtering rules.

    Example:
    Input: low cost contractor
    Output: cost < 50

    Input: high experience and safety
    Output: experience > 5 AND safety > 3

    Now:
    {query}
    """

    response = llm.invoke(prompt)
    return response.content

# -------------------------
# APPLY FILTERS
# -------------------------
def apply_filters(data, rules):
    filtered = []

    for d in data:
        try:
            if "cost < 50" in rules and d["cost"] >= 50:
                continue
            if "experience > 5" in rules and d["experience"] <= 5:
                continue
            if "safety > 3" in rules and d["safety"] <= 3:
                continue

            filtered.append(d)

        except:
            continue

    return filtered

# -------------------------
# DISPLAY CHAT HISTORY
# -------------------------
for msg in st.session_state.messages:
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])

# -------------------------
# USER INPUT
# -------------------------
user_input = st.chat_input("Ask like: low cost contractor with high safety")

if user_input:

    st.session_state.messages.append({"role": "user", "content": user_input})

    with st.chat_message("user"):
        st.markdown(user_input)

    if not st.session_state.data:
        response = "⚠️ Please upload contractor PDFs first."

    else:
        data = st.session_state.data

        # AI interprets query
        rules = interpret_query(user_input)

        # Apply filters
        filtered = apply_filters(data, rules)

        if not filtered:
            filtered = data  # fallback

        # Score
        for d in filtered:
            d["score"] = calculate_score(d)

        ranked = sorted(filtered, key=lambda x: x["score"], reverse=True)

        if not ranked:
            response = "❌ No contractors found."

        else:
            best = ranked[0]

            response = f"""
### 🤖 AI Analysis

**Understood Requirements:**  
{rules}

---

### ✅ Best Contractor: **{best['name']}**
Score: {best['score']}

---

### 📊 Ranking:
"""

            for r in ranked:
                response += f"- {r['name']} → {r['score']}\n"

    with st.chat_message("assistant"):
        st.markdown(response)

    st.session_state.messages.append({"role": "assistant", "content": response})