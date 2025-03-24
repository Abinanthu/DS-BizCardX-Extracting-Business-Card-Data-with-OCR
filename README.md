# DS-BizCardX-Extracting-Business-Card-Data-with-OCR
This Python code provides a solution for extracting and storing business card data using Optical Character Recognition (OCR) and a web app interface built with Streamlit. Here's a brief description of the code:
Key Components:
Imports:

import pyodbc  # Import pyodbc for SQL Server connection
import easyocr as ocr  # OCR
import streamlit as st  # Web App
from PIL import Image  # Image Processing
import numpy as np  # Image Processing 
import re
import io

# Set Streamlit layout to wide
st.set_page_config(layout="wide")

# Getting Secrets from Streamlit Secret File
SQLServer_server = "LENOVO-V130"  # SQL Server hostname or IP
SQLServer_database = "YoutubeDataHarsvesting"

# Connect to SQL Server using pyodbc with Windows Authentication
connection = pyodbc.connect(
    driver='{ODBC Driver 17 for SQL Server}',
    server=SQLServer_server,
    database=SQLServer_database,
    trusted_connection='yes'  # Use Windows Authentication
)

cursor = connection.cursor()

# Title formatting function
def format_title(title: str):
    """
    Formats the given title with a colored box and padding
    """
    formatted_title = f"<div style='padding:10px;background-color:rgb(230, 0, 172, 0.3);border-radius:10px'><h1 style='color:rgb(204, 0, 153);text-align:center;'>{title}</h1></div>"
    return formatted_title

# Use the function to format your title
st.markdown(format_title("UNLOCKING DATA FROM BUSINESS CARDS USING OCR"), unsafe_allow_html=True)

# Empty space for layout
st.write(" ")
st.write(" ")
st.write(" ")
st.write("### UPLOAD ANY BUSINESS CARD IMAGE TO EXTRACT INFORMATION ")

# Streamlit layout setup
CD, col1, col2, col3 = st.columns([0.5, 4, 1, 4])

with col1:
    # Image uploader
    st.write("#### SELECT IMAGE")
    image = st.file_uploader(label="", type=['png', 'jpg', 'jpeg'])

@st.cache
def load_model(): 
    reader = ocr.Reader(['en'])
    return reader 

reader = load_model()  # Load OCR model

if image is not None:
    input_image = Image.open(image)  # Read image
    with col1:
        st.image(input_image)  # Display image  
        st.write(" ")

    result = reader.readtext(np.array(input_image))
    result_text = []  # Empty list for results
    for text in result:
        result_text.append(text[1])

    PH = []
    PHID = []  
    ADD = set()
    AID = []
    EMAIL = ''
    EID = ''
    PIN = ''
    PID = ''
    WEB = ''
    WID = ''
    
    # Extract information from OCR result
    for i, string in enumerate(result_text):   
        # TO FIND EMAIL
        if re.search(r'@', string.lower()):
            EMAIL = string.lower()
            EID = i
        
        # TO FIND PINCODE
        match = re.search(r'\d{6,7}', string.lower())
        if match:
            PIN = match.group()
            PID = i
                        
        # TO FIND PHONE NUMBER    
        match = re.search(r'(?:ph|phone|phno)?\s*(?:[+-]?\d\s*[\(\)]*){7,}', string)
        if match and len(re.findall(r'\d', string)) > 7:
            PH.append(string)
            PHID.append(i)
            
        # TO FIND ADDRESS 
        keywords = ['road', 'floor', ' st ', 'st,', 'street', ' dt ', 'district',
                    'near', 'beside', 'opposite', ' at ', ' in ', 'center', 'main road',
                   'state','country', 'post','zip','city','zone','mandal','town','rural',
                    'circle','next to','across from','area','building','towers','village',
                    ' ST ',' VA ',' VA,',' EAST ',' WEST ',' NORTH ',' SOUTH ']
        # Define the regular expression pattern to match six or seven continuous digits
        digit_pattern = r'\d{6,7}'
        # Check if the string contains any of the keywords or a sequence of six or seven digits
        if any(keyword in string.lower() for keyword in keywords) or re.search(digit_pattern, string):
            ADD.add(string)
            AID.append(i)
            
        # TO FIND STATE (USING SIMILARITY SCORE)
        states = ['Andhra Pradesh', 'Arunachal Pradesh', 'Assam', 'Bihar', 'Chhattisgarh', 'Goa', 'Gujarat', 
                  'Haryana','Hyderabad', 'Himachal Pradesh', 'Jharkhand', 'Karnataka', 'Kerala', 'Madhya Pradesh',
                  'Maharashtra', 'Manipur', 'Meghalaya', 'Mizoram', 'Nagaland', 'Odisha', 'Punjab', 
                  'Rajasthan', 'Sikkim', 'Tamil Nadu', 'Telangana', 'Tripura', 'Uttar Pradesh', 'Uttarakhand', 'West Bengal',
                  "United States", "China", "Japan", "Germany", "United Kingdom", "France", "India", 
                  "Canada", "Italy", "South Korea", "Russia", "Australia", "Brazil", "Spain", "Mexico", 'USA','UK']
        
        import Levenshtein
        def string_similarity(s1, s2):
            distance = Levenshtein.distance(s1, s2)
            similarity = 1 - (distance / max(len(s1), len(s2)))
            return similarity * 100
        
        for x in states:
            similarity = string_similarity(x.lower(), string.lower())
            if similarity > 50:
                ADD.add(string)
                AID.append(i)
                
        # WEBSITE URL          
        if re.match(r"(?!.*@)(www|.*com$)", string):
            WEB = string.lower()
            WID = i 
            
    with col3: 
        # DISPLAY ALL THE EXTRACTED INFORMATION
        st.write("#### EXTRACTED TEXT")
        st.write('##### :blue[WEBSITE URL: ] '+ str(WEB))
        st.write('##### :blue[EMAIL: ] '+ str(EMAIL)) 
        st.write('##### :blue[PIN CODE: ] '+ str(PIN)) 
        ph_str = ', '.join(PH)
        st.write('##### :blue[PHONE NUMBER(S): ] '+ ph_str)
        add_str = ' '.join([str(elem) for elem in ADD])
        st.write('##### :blue[ADDRESS: ] ', add_str)

        IDS = [EID, PID, WID]
        IDS.extend(AID)
        IDS.extend(PHID)

        oth = ''                               
        fin = []                         
        for i, string in enumerate(result_text):
            if i not in IDS:
                if len(string) >= 4 and ',' not in string and '.' not in string and 'www.' not in string:
                    if not re.match("^[0-9]{0,3}$", string) and not re.match("^[^a-zA-Z0-9]+$", string):
                        numbers = re.findall(r'\d+', string)
                        if len(numbers) == 0 or all(len(num) < 3 for num in numbers) and not any(num in string for num in ['0','1','2','3','4','5','6','7','8','9']*3):
                            fin.append(string)
        st.write('##### :blue[CARD HOLDER & COMPANY DETAILS: ] ')
        for i in fin:
            st.write('##### '+i)

        UP = st.button('UPLOAD TO DATABASE', key=90)
                    
        # DATABASE CODE
        website = str(WEB)
        email = str(EMAIL)
        pincode = str(PIN)
        phoneno = ph_str
        address = add_str
        det_str = ' '.join([str(elem) for elem in fin])
        details = det_str
        image.seek(0)
        image_data = image.read()

        # IF UPLOAD BUTTON IS PRESSED, UPLOAD TO DATABASE
        if UP:
            if image is not None:
                # Read image data
                # Insert image data into SQL Server database
                data = (website, email, pincode, phoneno, address, details, image_data)
                sql = """INSERT INTO business_cards 
                         (website_url, email, pin_code, phone_number, address_line, holder_details, business_card_photo) 
                         VALUES (?, ?, ?, ?, ?, ?, ?)"""
                cursor.execute(sql, data)
                connection.commit()
            else:
                st.write('Please upload business card')

st.write(' ')
st.write(' ')

col1.markdown("<style>div[data-testid='stHorizontalBlock'] { background-color: rgb(230, 0, 172, 0.1); }</style>", unsafe_allow_html=True)

# DATABASE PART
st.write('### EXPLORE BUSINESS CARDS DATABASE ')
cd, c1, c2, c3 = st.columns([0.5, 4, 1, 4])

with c1: 
    st.write(' ')
    st.write("#### BUSINESS CARDS AVAILABLE IN DATABASE")
    cursor.execute("SELECT id FROM business_cards")
    rows = cursor.fetchall()
    l = []
    # DISPLAY ALL THE CARDS AS BUTTONS
    for row in rows:
        l.append(row[0])
        button_label = f"SHOW BUSINESS CARD: {row[0]}"
        if st.button(button_label):
            cursor.execute("SELECT * FROM business_cards WHERE id = " + str(row[0]))
            row1 = cursor.fetchone()
            website_url = row1[1]
            email = row1[2]
            pin_code = row1[3]
            phone_number = row1[4]  # Adjusted column name
            address = row1[5]
            holder_details = row1[6]

            # DISPLAY SELECTED CARD DETAILS
            st.write('##### :blue[WEBSITE URL: ] '+ str(website_url))
            st.write('##### :blue[EMAIL: ] '+ str(email))
            st.write('##### :blue[PIN CODE: ] '+ str(pin_code))
            st.write('##### :blue[PHONE NUMBER(S): ] '+ str(phone_number))
            st.write('##### :blue[ADDRESS: ] '+ str(address))
            st.write('##### :blue[CARD HOLDER DETAILS: ] '+ str(holder_details))
            st.write(' ')
            st.write(' ')   
