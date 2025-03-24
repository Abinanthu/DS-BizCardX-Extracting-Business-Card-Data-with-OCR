# DS-BizCardX-Extracting-Business-Card-Data-with-OCR
This Python code provides a solution for extracting and storing business card data using Optical Character Recognition (OCR) and a web app interface built with Streamlit. Here's a brief description of the code:
Key Components:
Imports:

pyodbc: Used for connecting to a SQL Server database.

easyocr: OCR library used to extract text from business card images.

streamlit: Web framework for building the user interface.

PIL (Pillow): Image processing library to handle image uploads.

re: Regular expressions for text processing.

numpy: Used to process the image data for OCR.

Streamlit Setup:

The app is configured to display content in a wide layout using st.set_page_config(layout="wide").

The user can upload an image of a business card, and the app will extract information like phone numbers, email, address, website, and more.

OCR Process:

The uploaded image is processed using the easyocr.Reader to extract text from the image.

The extracted text is analyzed using regular expressions to find specific details like:

Phone Numbers: Detected using regex for common phone formats.

Email: Extracted by searching for the "@" symbol.

Pin Codes: Extracted by searching for digit patterns.

Address: Identified using keywords related to locations (e.g., road, state, city).

Website URL: Detected using regex for common URL formats.

Card Holder & Company Details: The remaining text is considered as card holder and company information.

Database Insertion:

The extracted information is stored in a SQL Server database if the "UPLOAD TO DATABASE" button is pressed. It stores:

Website URL, email, phone number(s), address, and card holder/company details.

The business card image is stored as binary data in the database.

Displaying Extracted Data:

The app displays the extracted information (e.g., website, email, phone numbers) in the UI.

It also allows the user to upload multiple business cards and view stored data from the database.

Database Interface:

The app allows users to explore business cards already stored in the database. Users can view the details of any card stored in the database by clicking on a "SHOW BUSINESS CARD" button.

Workflow:
The user uploads a business card image.

The OCR library extracts the text from the image.

The code processes the extracted text to identify key pieces of information like phone numbers, email, address, etc.

If the user clicks "UPLOAD TO DATABASE", the information is inserted into the SQL Server database.

The user can explore the stored business card records by querying the database and viewing the details.

This solution enables a simple way to extract, store, and display business card information using OCR and a web interface.
