# Ayush
pip install streamlit pandas openpyxl PyPDF2 textblob spacy
# query_corrector.py

from textblob import TextBlob

def correct_query(query):
    corrected = str(TextBlob(query).correct())
    return corrected
    # intent_detector.py

def detect_intent(query):

    query = query.lower()

    if "summary" in query or "summarize" in query:
        return "Summary"

    elif "count" in query:
        return "Count"

    elif "average" in query or "mean" in query:
        return "Aggregation"

    elif "highest" in query or "lowest" in query:
        return "Analytics"

    elif "sort" in query:
        return "Sorting"

    elif "find" in query or "search" in query:
        return "Search"

    elif "filter" in query:
        return "Filter"

    return "Unknown"
    # pdf_processor.py

import PyPDF2

def read_pdf(file):

    text = ""

    reader = PyPDF2.PdfReader(file)

    for page in reader.pages:
        text += page.extract_text()

    return text
    # excel_processor.py

import pandas as pd

def read_spreadsheet(file):

    if file.name.endswith(".csv"):
        return pd.read_csv(file)

    return pd.read_excel(file)
    # app.py

import streamlit as st
import pandas as pd

from pdf_processor import read_pdf
from excel_processor import read_spreadsheet
from query_corrector import correct_query
from intent_detector import detect_intent

st.title("Document Intelligence System")

uploaded_file = st.file_uploader(
    "Upload PDF/Excel/CSV",
    type=["pdf","xlsx","csv"]
)

query = st.text_input("Enter Query")

if uploaded_file and query:

    corrected_query = correct_query(query)

    intent = detect_intent(corrected_query)

    file_type = uploaded_file.name.split(".")[-1]

    result = ""

    if file_type == "pdf":

        text = read_pdf(uploaded_file)

        if intent == "Summary":
            result = text[:1000]

        elif intent == "Search":
            result = "Keyword Found" if corrected_query.split()[-1] in text else "Not Found"

    else:

        df = read_spreadsheet(uploaded_file)

        if intent == "Count":
            result = len(df)

        elif intent == "Aggregation":

            numeric_cols = df.select_dtypes(include='number')

            result = numeric_cols.mean()

        elif intent == "Sorting":

            result = df.sort_values(df.columns[0])

        elif intent == "Analytics":

            result = df.describe()

    st.subheader("Output")

    st.write("File Type:", file_type)
    st.write("Original Query:", query)
    st.write("Corrected Query:", corrected_query)
    st.write("Detected Intent:", intent)
    st.write("Result:")
    st.write(result)
