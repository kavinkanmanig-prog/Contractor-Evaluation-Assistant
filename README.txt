PROJECT TITLE:
CMRL Contractor Evaluation System (AI + Chatbot)

DESCRIPTION:
This project is an AI-based system designed to evaluate contractors based on their company data provided in PDF format.

The system allows users to upload contractor documents, analyze them, and identify the best contractor using scoring logic and AI-based query understanding.

FEATURES:
- Upload multiple contractor PDFs
- Chatbot-style interface
- Natural language query processing
- Contractor scoring and ranking
- Best contractor recommendation

TECHNOLOGIES USED:
- Python
- Streamlit (UI)
- Ollama (LLM for query understanding)
- Rule-based scoring system

WORKFLOW:
1. Upload contractor PDFs
2. Extract structured data
3. Interpret user query using AI
4. Apply filters
5. Score contractors
6. Display ranked results

HOW TO RUN:
1. Install requirements:
   pip install -r requirements.txt

2. Run the application:
   streamlit run app.py

3. Open browser:
   http://localhost:8501

4. Upload sample PDFs and test queries

EXAMPLE QUERIES:
- low cost contractor
- high experience and safety
- best contractor

FUTURE IMPROVEMENTS:
- Real-time contractor database integration
- Advanced AI-based decision making
- Dashboard visualization