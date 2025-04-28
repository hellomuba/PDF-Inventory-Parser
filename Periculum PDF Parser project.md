# Periculum DS Internship - Technical Assessment - Mubarak Ibrahim

This notebook implements a data pipeline for processing home inventory PDF files.

My approach:
1. First of all, I Extract raw text from PDF
2. Align content line by line
3. Extract structured data
4. And finally, Output the results as JSON

## Firstly I installed the Pypdf2 package and  Import Required Libraries # 


```python
pip install PyPDF2 
```

    Requirement already satisfied: PyPDF2 in c:\users\xtamaliy\anaconda3\lib\site-packages (3.0.1)
    Requirement already satisfied: typing_extensions>=3.10.0.0 in c:\users\xtamaliy\anaconda3\lib\site-packages (from PyPDF2) (4.1.1)
    Note: you may need to restart the kernel to use updated packages.
    


```python

import json
from datetime import datetime
import PyPDF2  # I have already install with this as above : pip install PyPDF2
import PyPDF2 as pdf # here i import the package 
from datetime import datetime
from PyPDF2 import PdfReader, PdfWriter
```


```python
dir(pdf)
```




    ['DocumentInformation',
     'PageObject',
     'PageRange',
     'PaperSize',
     'PasswordType',
     'PdfFileMerger',
     'PdfFileReader',
     'PdfFileWriter',
     'PdfMerger',
     'PdfReader',
     'PdfWriter',
     'Transformation',
     '__all__',
     '__builtins__',
     '__cached__',
     '__doc__',
     '__file__',
     '__loader__',
     '__name__',
     '__package__',
     '__path__',
     '__spec__',
     '__version__',
     '__warningregistry__',
     '_cmap',
     '_codecs',
     '_encryption',
     '_merger',
     '_page',
     '_protocols',
     '_reader',
     '_security',
     '_utils',
     '_version',
     '_writer',
     'constants',
     'errors',
     'filters',
     'generic',
     'pagerange',
     'papersizes',
     'parse_filename_page_ranges',
     'types',
     'warnings',
     'xmp']




```python
pdf.__version__ #just checking for the latest version from the documentation
```




    '3.0.1'




```python
# to get the fil"
pdf_path = "home_inventory.pdf"

file = open(pdf_path, "rb")
reader = PdfReader(file)
```


```python
info = reader.metadata # to read the document information
print(info)
```

    {'/Author': 'pc', '/CreationDate': 'D:20250425205051Z', '/Creator': 'Microsoft® Excel® 2019', '/ModDate': "D:20250425215148+01'00'", '/Producer': 'Adobe PDF Services'}
    


```python
info.author

```




    'pc'




```python
info['/CreationDate']
```




    'D:20250425205051Z'




```python
len(reader.pages) #to check the total of pages
```




    10



## Define Classes

As required, we need two classes:
- OwnerInfo for storing owner details
- Inventory for storing inventory items


```python
class OwnerInfo:
    def __init__(self, owner_name="", owner_address="", owner_telephone=""):
        self.owner_name = owner_name
        self.owner_address = owner_address
        self.owner_telephone = owner_telephone
    
    def to_dict(self):
        # here to convert to dictionary for JSON
        return {
            "owner_name": self.owner_name,
            "owner_address": self.owner_address,
            "owner_telephone": self.owner_telephone
        }
```


```python
class Inventory:
    def __init__(self, purchase_date="", serial_number="", description="", source_style_area="", value=""):
        # Initialize inventory item attributes
        self.purchase_date = purchase_date
        self.serial_number = serial_number 
        self.description = description
        self.source_style_area = source_style_area
        self.value = value
    
    def to_dict(self): # Also to Convert to dictionary in order to serialize the data in JSON format
        return {
            "purchase_date": self.purchase_date,
            "serial_number": self.serial_number,
            "description": self.description,
            "source_style_area": self.source_style_area,
            "value": self.value
        }
```

## PDF Processing Functions

Now let's implement the required functions step by step.


```python
def get_data_from_pdf(pdf_path):
    """Extracts raw text from a PDF file."""
    text = ""
    
    try:
        with open(pdf_path, 'rb') as file:
            # Create a PDF reader object
            reader = PyPDF2.PdfReader(file)
            
            # Iterate through each page and extract text
            for page in reader.pages:
                text += page.extract_text()
                
    except Exception as e:
        print(f"Error reading PDF: {e}")
        
    return text
get_data_from_pdf(pdf_path)
```




    'S/N Area Item Description Source Purchase Date Style Serial No Value\n1Living Room Desk Target 07/06/2018 Premium 6DDZ7S36 846.59 $        \n2Kitchen LED TV Walmart 31/05/2015 Classic NJEZ3OPO 382.04 $        \n3Living Room LED TV Target 03/03/2019 Premium HRIS4LI8 1,603.37$     \n4Garage Dining Table Wayfair 26/03/2023 Modern 2HLNMD64 552.74 $        \n5Living Room Tool Set Target 06/01/2020 Classic R08QDU0S 1,546.39$     \n6Office LED TV Amazon 03/04/2015 Classic GSABG41R 1,319.36$     \n7Office Mattress Amazon 28/02/2022 Premium DVTMS64O 573.10 $        \n8Bedroom Tool Set Home Depot 07/04/2017 Classic RB93WPTH 1,421.38$     \n9Dining Room Mattress Target 20/03/2017 Compact 7RX8YNW9 1,760.11$     \n10Living Room Desk Target 30/06/2019 Classic US1B0BQI 941.43 $        \n11Garage Dining Table Wayfair 09/02/2016 Compact 1J7088BO 71.12 $           \n12Bedroom Desk Home Depot 21/08/2025 Modern GVJFQYT1 841.41 $        \n13Office Dining Table Best Buy 23/09/2022 Modern 5EXQQOB1 1,686.96$     \n14Kitchen Dining Table Amazon 07/04/2017 Premium QZY31U1R 1,457.39$     \n15Garage Mattress Home Depot 28/05/2024 Classic D4FYUJ8E 182.96 $        \n16Garage Dining Table Amazon 13/08/2023 Premium KO6MGEIY 127.00 $        \n17Kitchen Tool Set Amazon 12/07/2022 Premium X7QIEB5T 1,457.28$     \n18Garage Desk Walmart 27/08/2024 Compact BI1UXOJ9 103.48 $        \n19Garage Dining Table Amazon 13/10/2023 Classic 4GEV5RRW 403.55 $        \n20Office LED TV Best Buy 12/12/2015 Classic F7IKU0DS 541.69 $        \n21Living Room LED TV Amazon 22/07/2017 Premium 8HCSY97G 1,851.93$     \n22Garage Tool Set Amazon 30/05/2024 Premium WOWMB806 795.76 $        \n23Living Room Stand Mixer Best Buy 25/07/2017 Premium 3BTI1E79 1,734.69$     \n24Living Room Tool Set Amazon 08/03/2016 Classic 7RBCLH8X 1,351.20$     \n25Bedroom Dining Table Home Depot 03/03/2020 Premium W5I85PVW 148.41 $        \n26Dining Room Desk Wayfair 25/11/2021 Premium YJHBPF1G 1,896.32$     \n27Living Room Dining Table Best Buy 16/05/2020 Modern GQPBD9OJ 974.74 $        \n28Garage Tool Set Amazon 03/02/2017 Classic 7WGHYRKQ 1,293.36$     \n29Garage Desk Walmart 10/09/2025 Modern 14TNDRC5 859.33 $        \n30Dining Room LED TV Best Buy 15/09/2018 Modern QDD9Q659 1,072.67$     \n31Bedroom LED TV Amazon 05/11/2024 Compact DXW026Z7 129.39 $        \n32Kitchen Tool Set Best Buy 15/11/2019 Compact GZFM295X 1,801.21$     \n33Dining Room Dining Table Amazon 15/02/2025 Classic 20SQB8Q2 284.67 $        \n34Dining Room LED TV Best Buy 18/03/2021 Compact XM9K7QGA 989.14 $        \n35Kitchen LED TV Target 06/09/2021 Classic LAFHPEDJ 1,009.04$     \n36Kitchen LED TV Walmart 17/12/2021 Premium ZR30ULCG 198.69 $        \n37Kitchen Desk Wayfair 09/12/2022 Classic LSCM9SW9 636.68 $        \n38Kitchen Mattress Wayfair 22/10/2020 Premium AJBJT7EX 1,640.52$     Portland, OR 97203\n(503) 555-1234HOME INVENTORY \nOwner Information\nJohn Doe\n123 Maple StreetName\nAddress\nCity, Zip, Address\nPhone Number39Garage Desk Amazon 02/12/2025 Premium AZ6CAP1P 1,688.91$     \n40Bedroom Mattress Best Buy 16/08/2021 Modern 9NENYCRT 351.21 $        \n41Bedroom LED TV Target 08/08/2023 Modern FGWHOPBV 545.70 $        \n42Living Room Dining Table Home Depot 02/12/2024 Modern 5SDT2DDH 1,069.33$     \n43Living Room LED TV Home Depot 31/12/2021 Compact GQYFK6DI 1,722.17$     \n44Dining Room Tool Set Amazon 19/08/2018 Classic 0L6NWX6I 1,070.79$     \n45Office Tool Set Walmart 26/08/2016 Modern UQZLEMM1 1,869.05$     \n46Office Tool Set Target 14/04/2016 Premium 4CWTZCN4 676.29 $        \n47Bedroom Stand Mixer Wayfair 26/10/2019 Compact NQZ5N4KF 1,383.84$     \n48Bedroom Tool Set Wayfair 02/02/2020 Compact 6CLD1BF1 1,126.46$     \n49Office Desk Target 05/04/2021 Modern WZLXX57B 1,875.80$     \n50Garage Dining Table Amazon 05/08/2025 Classic K4SZAM0B 1,691.50$     \n51Office Dining Table Wayfair 21/07/2019 Premium 61KZ1NEC 181.27 $        \n52Bedroom Stand Mixer Amazon 14/02/2022 Classic DKZRBZ17 1,082.50$     \n53Bedroom LED TV Best Buy 24/11/2022 Modern 90IVJLBV 1,540.82$     \n54Garage Desk Wayfair 21/02/2025 Classic 6D2JL5JM 126.71 $        \n55Dining Room Dining Table Wayfair 18/05/2016 Premium CDTEHXP4 214.22 $        \n56Living Room Desk Amazon 20/08/2019 Modern NVABJVMU 56.43 $           \n57Bedroom Tool Set Target 03/07/2020 Premium G6DIX222 1,754.68$     \n58Dining Room Dining Table Walmart 10/10/2025 Classic 24K5YVA6 1,480.14$     \n59Dining Room LED TV Best Buy 17/10/2025 Compact O25XJTTZ 1,752.55$     \n60Living Room Stand Mixer Home Depot 22/05/2021 Classic 52DKP5S8 494.35 $        \n61Dining Room Mattress Wayfair 13/03/2022 Premium ZGUTA8UY 1,594.36$     \n62Garage Tool Set Wayfair 25/05/2017 Premium SXUBR4HY 828.29 $        \n63Garage LED TV Target 09/09/2018 Modern 48HB1W2B 962.78 $        \n64Garage Mattress Best Buy 14/03/2021 Premium CCOHLZAG 1,615.59$     \n65Bedroom Tool Set Amazon 21/12/2018 Classic 991Y960E 1,423.63$     \n66Kitchen LED TV Target 09/02/2025 Premium 7I484XUK 479.91 $        \n67Garage Desk Target 24/12/2022 Classic P3UQ5WZJ 936.75 $        \n68Bedroom Stand Mixer Wayfair 07/03/2017 Compact RX9LWPCE 1,832.53$     \n69Office LED TV Amazon 29/01/2017 Premium UIYI0O9B 428.69 $        \n70Bedroom Dining Table Wayfair 12/10/2025 Premium UVCNH3UB 1,104.24$     \n71Living Room Stand Mixer Home Depot 02/11/2021 Modern SNXG9HAZ 1,287.57$     \n72Living Room Stand Mixer Walmart 18/08/2023 Premium NBF3C3UK 88.80 $           \n73Garage Mattress Amazon 11/01/2025 Premium WPPV80BC 1,611.58$     \n74Office Dining Table Target 21/07/2020 Modern WVW41IBW 545.46 $        \n75Garage Tool Set Best Buy 24/02/2020 Modern 47CEOIYC 1,227.18$     \n76Garage Stand Mixer Walmart 11/12/2015 Premium DYB3CBUZ 1,459.15$     \n77Dining Room Stand Mixer Home Depot 08/02/2023 Premium 59HTZVC7 443.69 $        \n78Office Stand Mixer Wayfair 10/02/2020 Premium UBRQK64K 1,751.47$     \n79Office Mattress Walmart 12/02/2023 Compact 1001E8B2 1,110.70$     \n80Office Tool Set Home Depot 11/10/2017 Classic 74MDGHEV 359.33 $        \n81Living Room Tool Set Amazon 23/01/2024 Compact 4DFZ4S7S 1,383.34$     \n82Bedroom Desk Home Depot 21/12/2025 Classic 2J91PY8I 102.34 $        \n83Office Mattress Amazon 23/01/2019 Classic IZQ6TDTE 1,279.92$     \n84Living Room Tool Set Walmart 22/11/2021 Compact HEGMBQY3 1,325.53$     \n85Bedroom Tool Set Home Depot 12/03/2018 Compact U5I6T0TB 791.40 $        86Office Desk Wayfair 09/08/2016 Compact UUIQVFOP 603.75 $        \n87Dining Room Mattress Walmart 19/06/2024 Premium X49NPQV7 1,069.49$     \n88Office Tool Set Wayfair 04/12/2017 Compact V4KM8SQ2 1,360.35$     \n89Office Dining Table Amazon 04/12/2023 Classic WVZC1KS8 1,646.35$     \n90Living Room LED TV Home Depot 28/05/2016 Classic 3X3UBT5G 407.54 $        \n91Kitchen Stand Mixer Home Depot 27/09/2021 Premium 7WPEMBLU 1,416.85$     \n92Office Stand Mixer Best Buy 02/04/2016 Modern SZP2ALMH 503.23 $        \n93Garage LED TV Amazon 06/09/2025 Premium C9DWHHXK 891.06 $        \n94Dining Room Mattress Best Buy 07/08/2019 Modern DP02GB8S 99.20 $           \n95Office LED TV Amazon 12/08/2022 Premium YOV6MOC4 403.41 $        \n96Office Stand Mixer Amazon 18/11/2015 Classic 8Y6ES9ZR 1,244.66$     \n97Office Tool Set Best Buy 30/03/2016 Premium R10N682J 1,752.79$     \n98Garage Mattress Wayfair 20/12/2024 Compact XYVY0DG6 130.41 $        \n99Kitchen Dining Table Walmart 27/11/2018 Compact 4X0CQUIA 1,374.61$     \n100 Dining Room Stand Mixer Best Buy 22/04/2021 Modern U7PXVQAB 278.45 $        \n101 Dining Room Tool Set Home Depot 04/07/2022 Modern 4OGYNJGI 250.32 $        \n102 Garage Desk Home Depot 11/05/2016 Premium Z9TTB4MF 800.91 $        \n103 Living Room Dining Table Home Depot 21/10/2017 Classic 7XW6R69R 912.83 $        \n104 Garage Desk Amazon 08/12/2017 Classic CW9LEOK5 579.18 $        \n105 Office Desk Best Buy 17/05/2023 Classic NBS18P9F 932.97 $        \n106 Kitchen Dining Table Wayfair 07/06/2021 Premium BYWR0VQ3 1,331.28$     \n107 Kitchen Tool Set Home Depot 24/07/2017 Compact Z4WKQAUD 1,726.77$     \n108 Bedroom Tool Set Walmart 11/06/2020 Modern QRYP4WVD 823.04 $        \n109 Dining Room Tool Set Amazon 10/08/2016 Modern 5DTN8WP1 1,364.42$     \n110 Kitchen Desk Best Buy 07/03/2015 Compact S1IZ2ORO 1,474.74$     \n111 Kitchen Tool Set Target 07/10/2022 Premium MSEPKFQG 327.57 $        \n112 Bedroom Mattress Target 28/09/2016 Classic V5T93UBK 50.75 $           \n113 Garage Dining Table Home Depot 29/04/2023 Compact AE0EWUCB 1,348.12$     \n114 Office Dining Table Target 28/10/2021 Compact RESWYF9C 91.72 $           \n115 Office Tool Set Target 12/02/2015 Modern GDKBSREF 745.94 $        \n116 Dining Room LED TV Wayfair 05/11/2018 Compact 4R596I04 520.17 $        \n117 Kitchen Dining Table Target 25/07/2019 Modern PU3JA7WI 855.00 $        \n118 Bedroom Mattress Target 10/02/2020 Premium CRUF4HVJ 1,564.71$     \n119 Living Room Mattress Amazon 18/08/2020 Compact 0N0SJGCY 1,384.72$     \n120 Dining Room Dining Table Amazon 16/09/2023 Classic V9W3YPMT 1,151.00$     \n121 Bedroom Stand Mixer Best Buy 31/05/2023 Premium ORNZN8KE 1,109.99$     \n122 Kitchen Mattress Home Depot 12/09/2019 Compact UR9LN2RL 138.31 $        \n123 Garage Stand Mixer Wayfair 24/09/2025 Compact IYGUXBJ8 273.55 $        \n124 Office Tool Set Wayfair 16/10/2016 Classic R0S5K5A7 324.30 $        \n125 Living Room Stand Mixer Wayfair 11/09/2025 Premium HJ62DG5N 738.16 $        \n126 Dining Room Tool Set Best Buy 27/10/2021 Classic 9WBR8ESA 988.11 $        \n127 Office Stand Mixer Target 15/12/2025 Classic O4QZM9OP 1,656.82$     \n128 Living Room Desk Amazon 08/02/2019 Premium KJQP8H26 1,386.27$     \n129 Dining Room Mattress Target 02/06/2015 Compact U8DVZFWV 571.46 $        \n130 Bedroom Tool Set Wayfair 13/04/2024 Modern BJR8OZOF 305.00 $        \n131 Office LED TV Home Depot 19/12/2017 Premium DENRQBQ6 1,219.77$     \n132 Office Dining Table Walmart 13/07/2015 Modern 2CZTWBEN 1,510.01$     133 Dining Room Stand Mixer Home Depot 21/09/2022 Compact 2A4RRKT4 1,676.31$     \n134 Kitchen LED TV Wayfair 21/05/2020 Premium ELLVSI6B 979.19 $        \n135 Office Mattress Walmart 20/03/2023 Classic WWMYJEAV 765.57 $        \n136 Garage Tool Set Wayfair 28/04/2021 Modern E8LNBEOM 630.40 $        \n137 Living Room Tool Set Wayfair 22/04/2017 Premium DCATHJ3H 1,576.72$     \n138 Kitchen LED TV Amazon 31/05/2021 Classic 0VA5WHK9 1,044.09$     \n139 Dining Room LED TV Amazon 09/10/2022 Premium JLKFBBS5 239.32 $        \n140 Bedroom Tool Set Wayfair 13/12/2018 Compact UW76JUEY 398.97 $        \n141 Dining Room LED TV Wayfair 29/11/2021 Compact WEVV8O3R 606.26 $        \n142 Living Room Dining Table Amazon 23/11/2020 Premium ENVSPMO9 515.32 $        \n143 Dining Room Stand Mixer Best Buy 04/12/2018 Compact MZ2LZRV5 638.99 $        \n144 Bedroom LED TV Best Buy 27/10/2017 Modern 7K0G5I95 1,677.74$     \n145 Bedroom Desk Walmart 23/01/2022 Compact MO66OFGP 787.01 $        \n146 Garage Tool Set Target 10/02/2017 Premium X8RLUG3J 1,438.51$     \n147 Bedroom Tool Set Best Buy 21/09/2021 Modern 55DBVC6I 1,464.33$     \n148 Office LED TV Walmart 04/03/2021 Premium BQ2K5ZP6 252.71 $        \n149 Dining Room LED TV Home Depot 04/01/2017 Compact 9OX4W1DP 865.35 $        \n150 Dining Room Dining Table Home Depot 04/08/2017 Classic 08RQPP9E 1,513.78$     \n151 Office LED TV Best Buy 22/10/2018 Compact BHQOC2AK 1,156.85$     \n152 Kitchen Stand Mixer Amazon 15/10/2022 Modern 27QA3NCY 511.88 $        \n153 Dining Room Stand Mixer Amazon 20/04/2017 Classic 828ZXE71 1,129.03$     \n154 Bedroom Dining Table Home Depot 02/01/2022 Classic K0M9NKF5 406.70 $        \n155 Kitchen LED TV Walmart 13/07/2019 Compact Y2O7L2B0 1,394.44$     \n156 Kitchen Desk Wayfair 07/02/2024 Modern NPS1GVZY 1,484.40$     \n157 Dining Room LED TV Amazon 12/06/2020 Modern E6DYBIWE 1,461.72$     \n158 Garage LED TV Walmart 21/01/2021 Compact ALOGE3QY 149.29 $        \n159 Bedroom LED TV Best Buy 11/12/2021 Modern LBS7UPM6 803.25 $        \n160 Office LED TV Wayfair 06/12/2019 Compact R5YCRXSQ 1,273.78$     \n161 Living Room Tool Set Target 13/04/2019 Premium NQKV79JA 1,750.27$     \n162 Garage LED TV Best Buy 13/05/2015 Compact P0ZY0J5A 601.50 $        \n163 Office LED TV Best Buy 15/03/2025 Premium U0EGB2H1 1,624.11$     \n164 Living Room Tool Set Walmart 20/03/2017 Premium T96GB8RE 1,873.35$     \n165 Kitchen Desk Target 28/09/2017 Premium IZ10KLG2 1,286.29$     \n166 Dining Room Desk Wayfair 24/10/2020 Classic EG70TSWM 1,747.33$     \n167 Kitchen LED TV Walmart 15/01/2020 Compact HUGYL1UA 415.92 $        \n168 Garage Dining Table Amazon 03/03/2018 Classic F17YVQ8P 528.30 $        \n169 Dining Room Tool Set Walmart 01/12/2018 Compact 2CH0K64T 1,247.92$     \n170 Living Room Stand Mixer Target 29/08/2019 Compact HYMC4KDB 179.10 $        \n171 Kitchen Tool Set Wayfair 24/10/2017 Premium A15U7SL6 752.20 $        \n172 Office Mattress Wayfair 01/06/2015 Classic 631PVQZ8 954.68 $        \n173 Living Room LED TV Amazon 06/09/2023 Compact LE7WASLW 1,227.77$     \n174 Living Room Mattress Amazon 06/11/2016 Compact AE0U85VB 181.98 $        \n175 Bedroom LED TV Best Buy 02/07/2019 Classic JMJI7NSB 417.10 $        \n176 Dining Room Stand Mixer Best Buy 06/07/2016 Classic R1FEM8QE 601.05 $        \n177 Bedroom LED TV Walmart 22/03/2019 Classic 83FL23MO 609.00 $        \n178 Dining Room Tool Set Home Depot 04/05/2015 Compact X89Z8FCM 1,632.71$     \n179 Kitchen LED TV Best Buy 15/11/2023 Premium MFRHLN1W 503.93 $        180 Garage LED TV Wayfair 19/12/2015 Premium I4N5J4VF 357.52 $        \n181 Office Dining Table Walmart 07/12/2022 Classic 81XY8BYB 1,880.87$     \n182 Dining Room Dining Table Wayfair 30/08/2016 Compact SFZM6UZB 1,240.03$     \n183 Bedroom Dining Table Wayfair 02/06/2023 Modern QK2VYRQ4 1,164.27$     \n184 Bedroom Dining Table Home Depot 10/07/2018 Compact MXCGC08A 325.25 $        \n185 Kitchen Desk Best Buy 09/08/2018 Classic WM2DI61E 190.63 $        \n186 Bedroom Dining Table Best Buy 26/04/2016 Classic UNFMT54I 1,176.83$     \n187 Garage Mattress Walmart 09/09/2022 Compact 55GAE48V 670.44 $        \n188 Bedroom Mattress Best Buy 26/04/2018 Modern R920ZMQD 1,474.32$     \n189 Office Tool Set Target 23/03/2020 Compact LTPEQFTI 153.32 $        \n190 Bedroom Desk Walmart 19/03/2024 Premium VB6ZKIQC 1,489.13$     \n191 Bedroom LED TV Walmart 15/07/2018 Compact 1HI6TR6A 1,741.16$     \n192 Garage Stand Mixer Walmart 16/04/2019 Premium DZCH7EWK 1,180.74$     Condition\nExcellent\nExcellent\nExcellent\nNew\nLike New\nLike New\nExcellent\nExcellent\nLike New\nLike New\nExcellent\nLike New\nExcellent\nGood\nNew\nExcellent\nExcellent\nNew\nLike New\nExcellent\nGood\nExcellent\nGood\nExcellent\nExcellent\nNew\nNew\nExcellent\nLike New\nLike New\nNew\nNew\nNew\nNew\nNew\nNew\nExcellent\nExcellentHOME INVENTORY Like New\nNew\nNew\nExcellent\nExcellent\nLike New\nExcellent\nLike New\nExcellent\nExcellent\nGood\nExcellent\nNew\nGood\nGood\nNew\nExcellent\nExcellent\nNew\nGood\nGood\nLike New\nNew\nExcellent\nExcellent\nGood\nGood\nExcellent\nLike New\nExcellent\nNew\nGood\nLike New\nLike New\nGood\nNew\nNew\nGood\nLike New\nGood\nExcellent\nGood\nLike New\nGood\nLike New\nExcellent\nNewGood\nLike New\nGood\nGood\nLike New\nLike New\nLike New\nLike New\nExcellent\nGood\nLike New\nExcellent\nGood\nExcellent\nLike New\nGood\nNew\nExcellent\nGood\nGood\nGood\nNew\nLike New\nGood\nExcellent\nNew\nNew\nLike New\nNew\nExcellent\nGood\nNew\nExcellent\nGood\nLike New\nLike New\nLike New\nLike New\nGood\nNew\nGood\nLike New\nGood\nNew\nNew\nLike New\nLike NewLike New\nNew\nExcellent\nExcellent\nLike New\nLike New\nGood\nLike New\nGood\nExcellent\nExcellent\nGood\nLike New\nGood\nGood\nGood\nNew\nNew\nLike New\nLike New\nLike New\nGood\nGood\nGood\nGood\nNew\nNew\nLike New\nExcellent\nNew\nLike New\nLike New\nGood\nNew\nGood\nLike New\nGood\nLike New\nGood\nLike New\nLike New\nNew\nExcellent\nNew\nGood\nGood\nExcellentNew\nExcellent\nNew\nGood\nGood\nGood\nLike New\nNew\nNew\nExcellent\nGood\nNew\nExcellent'




```python
raw_text = get_data_from_pdf(pdf_path)
```


```python
def align_content(raw_text): # to split the raw text content line by line.
    lines = raw_text.split('\n')
    clean_lines = []  # to remove empty lines and trim whitespace
    for line in lines:
        line = line.strip()
        if line:  # Only add non-empty lines
            clean_lines.append(line)
    
    return clean_lines
align_content(raw_text)
```




    ['S/N Area Item Description Source Purchase Date Style Serial No Value',
     '1Living Room Desk Target 07/06/2018 Premium 6DDZ7S36 846.59 $',
     '2Kitchen LED TV Walmart 31/05/2015 Classic NJEZ3OPO 382.04 $',
     '3Living Room LED TV Target 03/03/2019 Premium HRIS4LI8 1,603.37$',
     '4Garage Dining Table Wayfair 26/03/2023 Modern 2HLNMD64 552.74 $',
     '5Living Room Tool Set Target 06/01/2020 Classic R08QDU0S 1,546.39$',
     '6Office LED TV Amazon 03/04/2015 Classic GSABG41R 1,319.36$',
     '7Office Mattress Amazon 28/02/2022 Premium DVTMS64O 573.10 $',
     '8Bedroom Tool Set Home Depot 07/04/2017 Classic RB93WPTH 1,421.38$',
     '9Dining Room Mattress Target 20/03/2017 Compact 7RX8YNW9 1,760.11$',
     '10Living Room Desk Target 30/06/2019 Classic US1B0BQI 941.43 $',
     '11Garage Dining Table Wayfair 09/02/2016 Compact 1J7088BO 71.12 $',
     '12Bedroom Desk Home Depot 21/08/2025 Modern GVJFQYT1 841.41 $',
     '13Office Dining Table Best Buy 23/09/2022 Modern 5EXQQOB1 1,686.96$',
     '14Kitchen Dining Table Amazon 07/04/2017 Premium QZY31U1R 1,457.39$',
     '15Garage Mattress Home Depot 28/05/2024 Classic D4FYUJ8E 182.96 $',
     '16Garage Dining Table Amazon 13/08/2023 Premium KO6MGEIY 127.00 $',
     '17Kitchen Tool Set Amazon 12/07/2022 Premium X7QIEB5T 1,457.28$',
     '18Garage Desk Walmart 27/08/2024 Compact BI1UXOJ9 103.48 $',
     '19Garage Dining Table Amazon 13/10/2023 Classic 4GEV5RRW 403.55 $',
     '20Office LED TV Best Buy 12/12/2015 Classic F7IKU0DS 541.69 $',
     '21Living Room LED TV Amazon 22/07/2017 Premium 8HCSY97G 1,851.93$',
     '22Garage Tool Set Amazon 30/05/2024 Premium WOWMB806 795.76 $',
     '23Living Room Stand Mixer Best Buy 25/07/2017 Premium 3BTI1E79 1,734.69$',
     '24Living Room Tool Set Amazon 08/03/2016 Classic 7RBCLH8X 1,351.20$',
     '25Bedroom Dining Table Home Depot 03/03/2020 Premium W5I85PVW 148.41 $',
     '26Dining Room Desk Wayfair 25/11/2021 Premium YJHBPF1G 1,896.32$',
     '27Living Room Dining Table Best Buy 16/05/2020 Modern GQPBD9OJ 974.74 $',
     '28Garage Tool Set Amazon 03/02/2017 Classic 7WGHYRKQ 1,293.36$',
     '29Garage Desk Walmart 10/09/2025 Modern 14TNDRC5 859.33 $',
     '30Dining Room LED TV Best Buy 15/09/2018 Modern QDD9Q659 1,072.67$',
     '31Bedroom LED TV Amazon 05/11/2024 Compact DXW026Z7 129.39 $',
     '32Kitchen Tool Set Best Buy 15/11/2019 Compact GZFM295X 1,801.21$',
     '33Dining Room Dining Table Amazon 15/02/2025 Classic 20SQB8Q2 284.67 $',
     '34Dining Room LED TV Best Buy 18/03/2021 Compact XM9K7QGA 989.14 $',
     '35Kitchen LED TV Target 06/09/2021 Classic LAFHPEDJ 1,009.04$',
     '36Kitchen LED TV Walmart 17/12/2021 Premium ZR30ULCG 198.69 $',
     '37Kitchen Desk Wayfair 09/12/2022 Classic LSCM9SW9 636.68 $',
     '38Kitchen Mattress Wayfair 22/10/2020 Premium AJBJT7EX 1,640.52$     Portland, OR 97203',
     '(503) 555-1234HOME INVENTORY',
     'Owner Information',
     'John Doe',
     '123 Maple StreetName',
     'Address',
     'City, Zip, Address',
     'Phone Number39Garage Desk Amazon 02/12/2025 Premium AZ6CAP1P 1,688.91$',
     '40Bedroom Mattress Best Buy 16/08/2021 Modern 9NENYCRT 351.21 $',
     '41Bedroom LED TV Target 08/08/2023 Modern FGWHOPBV 545.70 $',
     '42Living Room Dining Table Home Depot 02/12/2024 Modern 5SDT2DDH 1,069.33$',
     '43Living Room LED TV Home Depot 31/12/2021 Compact GQYFK6DI 1,722.17$',
     '44Dining Room Tool Set Amazon 19/08/2018 Classic 0L6NWX6I 1,070.79$',
     '45Office Tool Set Walmart 26/08/2016 Modern UQZLEMM1 1,869.05$',
     '46Office Tool Set Target 14/04/2016 Premium 4CWTZCN4 676.29 $',
     '47Bedroom Stand Mixer Wayfair 26/10/2019 Compact NQZ5N4KF 1,383.84$',
     '48Bedroom Tool Set Wayfair 02/02/2020 Compact 6CLD1BF1 1,126.46$',
     '49Office Desk Target 05/04/2021 Modern WZLXX57B 1,875.80$',
     '50Garage Dining Table Amazon 05/08/2025 Classic K4SZAM0B 1,691.50$',
     '51Office Dining Table Wayfair 21/07/2019 Premium 61KZ1NEC 181.27 $',
     '52Bedroom Stand Mixer Amazon 14/02/2022 Classic DKZRBZ17 1,082.50$',
     '53Bedroom LED TV Best Buy 24/11/2022 Modern 90IVJLBV 1,540.82$',
     '54Garage Desk Wayfair 21/02/2025 Classic 6D2JL5JM 126.71 $',
     '55Dining Room Dining Table Wayfair 18/05/2016 Premium CDTEHXP4 214.22 $',
     '56Living Room Desk Amazon 20/08/2019 Modern NVABJVMU 56.43 $',
     '57Bedroom Tool Set Target 03/07/2020 Premium G6DIX222 1,754.68$',
     '58Dining Room Dining Table Walmart 10/10/2025 Classic 24K5YVA6 1,480.14$',
     '59Dining Room LED TV Best Buy 17/10/2025 Compact O25XJTTZ 1,752.55$',
     '60Living Room Stand Mixer Home Depot 22/05/2021 Classic 52DKP5S8 494.35 $',
     '61Dining Room Mattress Wayfair 13/03/2022 Premium ZGUTA8UY 1,594.36$',
     '62Garage Tool Set Wayfair 25/05/2017 Premium SXUBR4HY 828.29 $',
     '63Garage LED TV Target 09/09/2018 Modern 48HB1W2B 962.78 $',
     '64Garage Mattress Best Buy 14/03/2021 Premium CCOHLZAG 1,615.59$',
     '65Bedroom Tool Set Amazon 21/12/2018 Classic 991Y960E 1,423.63$',
     '66Kitchen LED TV Target 09/02/2025 Premium 7I484XUK 479.91 $',
     '67Garage Desk Target 24/12/2022 Classic P3UQ5WZJ 936.75 $',
     '68Bedroom Stand Mixer Wayfair 07/03/2017 Compact RX9LWPCE 1,832.53$',
     '69Office LED TV Amazon 29/01/2017 Premium UIYI0O9B 428.69 $',
     '70Bedroom Dining Table Wayfair 12/10/2025 Premium UVCNH3UB 1,104.24$',
     '71Living Room Stand Mixer Home Depot 02/11/2021 Modern SNXG9HAZ 1,287.57$',
     '72Living Room Stand Mixer Walmart 18/08/2023 Premium NBF3C3UK 88.80 $',
     '73Garage Mattress Amazon 11/01/2025 Premium WPPV80BC 1,611.58$',
     '74Office Dining Table Target 21/07/2020 Modern WVW41IBW 545.46 $',
     '75Garage Tool Set Best Buy 24/02/2020 Modern 47CEOIYC 1,227.18$',
     '76Garage Stand Mixer Walmart 11/12/2015 Premium DYB3CBUZ 1,459.15$',
     '77Dining Room Stand Mixer Home Depot 08/02/2023 Premium 59HTZVC7 443.69 $',
     '78Office Stand Mixer Wayfair 10/02/2020 Premium UBRQK64K 1,751.47$',
     '79Office Mattress Walmart 12/02/2023 Compact 1001E8B2 1,110.70$',
     '80Office Tool Set Home Depot 11/10/2017 Classic 74MDGHEV 359.33 $',
     '81Living Room Tool Set Amazon 23/01/2024 Compact 4DFZ4S7S 1,383.34$',
     '82Bedroom Desk Home Depot 21/12/2025 Classic 2J91PY8I 102.34 $',
     '83Office Mattress Amazon 23/01/2019 Classic IZQ6TDTE 1,279.92$',
     '84Living Room Tool Set Walmart 22/11/2021 Compact HEGMBQY3 1,325.53$',
     '85Bedroom Tool Set Home Depot 12/03/2018 Compact U5I6T0TB 791.40 $        86Office Desk Wayfair 09/08/2016 Compact UUIQVFOP 603.75 $',
     '87Dining Room Mattress Walmart 19/06/2024 Premium X49NPQV7 1,069.49$',
     '88Office Tool Set Wayfair 04/12/2017 Compact V4KM8SQ2 1,360.35$',
     '89Office Dining Table Amazon 04/12/2023 Classic WVZC1KS8 1,646.35$',
     '90Living Room LED TV Home Depot 28/05/2016 Classic 3X3UBT5G 407.54 $',
     '91Kitchen Stand Mixer Home Depot 27/09/2021 Premium 7WPEMBLU 1,416.85$',
     '92Office Stand Mixer Best Buy 02/04/2016 Modern SZP2ALMH 503.23 $',
     '93Garage LED TV Amazon 06/09/2025 Premium C9DWHHXK 891.06 $',
     '94Dining Room Mattress Best Buy 07/08/2019 Modern DP02GB8S 99.20 $',
     '95Office LED TV Amazon 12/08/2022 Premium YOV6MOC4 403.41 $',
     '96Office Stand Mixer Amazon 18/11/2015 Classic 8Y6ES9ZR 1,244.66$',
     '97Office Tool Set Best Buy 30/03/2016 Premium R10N682J 1,752.79$',
     '98Garage Mattress Wayfair 20/12/2024 Compact XYVY0DG6 130.41 $',
     '99Kitchen Dining Table Walmart 27/11/2018 Compact 4X0CQUIA 1,374.61$',
     '100 Dining Room Stand Mixer Best Buy 22/04/2021 Modern U7PXVQAB 278.45 $',
     '101 Dining Room Tool Set Home Depot 04/07/2022 Modern 4OGYNJGI 250.32 $',
     '102 Garage Desk Home Depot 11/05/2016 Premium Z9TTB4MF 800.91 $',
     '103 Living Room Dining Table Home Depot 21/10/2017 Classic 7XW6R69R 912.83 $',
     '104 Garage Desk Amazon 08/12/2017 Classic CW9LEOK5 579.18 $',
     '105 Office Desk Best Buy 17/05/2023 Classic NBS18P9F 932.97 $',
     '106 Kitchen Dining Table Wayfair 07/06/2021 Premium BYWR0VQ3 1,331.28$',
     '107 Kitchen Tool Set Home Depot 24/07/2017 Compact Z4WKQAUD 1,726.77$',
     '108 Bedroom Tool Set Walmart 11/06/2020 Modern QRYP4WVD 823.04 $',
     '109 Dining Room Tool Set Amazon 10/08/2016 Modern 5DTN8WP1 1,364.42$',
     '110 Kitchen Desk Best Buy 07/03/2015 Compact S1IZ2ORO 1,474.74$',
     '111 Kitchen Tool Set Target 07/10/2022 Premium MSEPKFQG 327.57 $',
     '112 Bedroom Mattress Target 28/09/2016 Classic V5T93UBK 50.75 $',
     '113 Garage Dining Table Home Depot 29/04/2023 Compact AE0EWUCB 1,348.12$',
     '114 Office Dining Table Target 28/10/2021 Compact RESWYF9C 91.72 $',
     '115 Office Tool Set Target 12/02/2015 Modern GDKBSREF 745.94 $',
     '116 Dining Room LED TV Wayfair 05/11/2018 Compact 4R596I04 520.17 $',
     '117 Kitchen Dining Table Target 25/07/2019 Modern PU3JA7WI 855.00 $',
     '118 Bedroom Mattress Target 10/02/2020 Premium CRUF4HVJ 1,564.71$',
     '119 Living Room Mattress Amazon 18/08/2020 Compact 0N0SJGCY 1,384.72$',
     '120 Dining Room Dining Table Amazon 16/09/2023 Classic V9W3YPMT 1,151.00$',
     '121 Bedroom Stand Mixer Best Buy 31/05/2023 Premium ORNZN8KE 1,109.99$',
     '122 Kitchen Mattress Home Depot 12/09/2019 Compact UR9LN2RL 138.31 $',
     '123 Garage Stand Mixer Wayfair 24/09/2025 Compact IYGUXBJ8 273.55 $',
     '124 Office Tool Set Wayfair 16/10/2016 Classic R0S5K5A7 324.30 $',
     '125 Living Room Stand Mixer Wayfair 11/09/2025 Premium HJ62DG5N 738.16 $',
     '126 Dining Room Tool Set Best Buy 27/10/2021 Classic 9WBR8ESA 988.11 $',
     '127 Office Stand Mixer Target 15/12/2025 Classic O4QZM9OP 1,656.82$',
     '128 Living Room Desk Amazon 08/02/2019 Premium KJQP8H26 1,386.27$',
     '129 Dining Room Mattress Target 02/06/2015 Compact U8DVZFWV 571.46 $',
     '130 Bedroom Tool Set Wayfair 13/04/2024 Modern BJR8OZOF 305.00 $',
     '131 Office LED TV Home Depot 19/12/2017 Premium DENRQBQ6 1,219.77$',
     '132 Office Dining Table Walmart 13/07/2015 Modern 2CZTWBEN 1,510.01$     133 Dining Room Stand Mixer Home Depot 21/09/2022 Compact 2A4RRKT4 1,676.31$',
     '134 Kitchen LED TV Wayfair 21/05/2020 Premium ELLVSI6B 979.19 $',
     '135 Office Mattress Walmart 20/03/2023 Classic WWMYJEAV 765.57 $',
     '136 Garage Tool Set Wayfair 28/04/2021 Modern E8LNBEOM 630.40 $',
     '137 Living Room Tool Set Wayfair 22/04/2017 Premium DCATHJ3H 1,576.72$',
     '138 Kitchen LED TV Amazon 31/05/2021 Classic 0VA5WHK9 1,044.09$',
     '139 Dining Room LED TV Amazon 09/10/2022 Premium JLKFBBS5 239.32 $',
     '140 Bedroom Tool Set Wayfair 13/12/2018 Compact UW76JUEY 398.97 $',
     '141 Dining Room LED TV Wayfair 29/11/2021 Compact WEVV8O3R 606.26 $',
     '142 Living Room Dining Table Amazon 23/11/2020 Premium ENVSPMO9 515.32 $',
     '143 Dining Room Stand Mixer Best Buy 04/12/2018 Compact MZ2LZRV5 638.99 $',
     '144 Bedroom LED TV Best Buy 27/10/2017 Modern 7K0G5I95 1,677.74$',
     '145 Bedroom Desk Walmart 23/01/2022 Compact MO66OFGP 787.01 $',
     '146 Garage Tool Set Target 10/02/2017 Premium X8RLUG3J 1,438.51$',
     '147 Bedroom Tool Set Best Buy 21/09/2021 Modern 55DBVC6I 1,464.33$',
     '148 Office LED TV Walmart 04/03/2021 Premium BQ2K5ZP6 252.71 $',
     '149 Dining Room LED TV Home Depot 04/01/2017 Compact 9OX4W1DP 865.35 $',
     '150 Dining Room Dining Table Home Depot 04/08/2017 Classic 08RQPP9E 1,513.78$',
     '151 Office LED TV Best Buy 22/10/2018 Compact BHQOC2AK 1,156.85$',
     '152 Kitchen Stand Mixer Amazon 15/10/2022 Modern 27QA3NCY 511.88 $',
     '153 Dining Room Stand Mixer Amazon 20/04/2017 Classic 828ZXE71 1,129.03$',
     '154 Bedroom Dining Table Home Depot 02/01/2022 Classic K0M9NKF5 406.70 $',
     '155 Kitchen LED TV Walmart 13/07/2019 Compact Y2O7L2B0 1,394.44$',
     '156 Kitchen Desk Wayfair 07/02/2024 Modern NPS1GVZY 1,484.40$',
     '157 Dining Room LED TV Amazon 12/06/2020 Modern E6DYBIWE 1,461.72$',
     '158 Garage LED TV Walmart 21/01/2021 Compact ALOGE3QY 149.29 $',
     '159 Bedroom LED TV Best Buy 11/12/2021 Modern LBS7UPM6 803.25 $',
     '160 Office LED TV Wayfair 06/12/2019 Compact R5YCRXSQ 1,273.78$',
     '161 Living Room Tool Set Target 13/04/2019 Premium NQKV79JA 1,750.27$',
     '162 Garage LED TV Best Buy 13/05/2015 Compact P0ZY0J5A 601.50 $',
     '163 Office LED TV Best Buy 15/03/2025 Premium U0EGB2H1 1,624.11$',
     '164 Living Room Tool Set Walmart 20/03/2017 Premium T96GB8RE 1,873.35$',
     '165 Kitchen Desk Target 28/09/2017 Premium IZ10KLG2 1,286.29$',
     '166 Dining Room Desk Wayfair 24/10/2020 Classic EG70TSWM 1,747.33$',
     '167 Kitchen LED TV Walmart 15/01/2020 Compact HUGYL1UA 415.92 $',
     '168 Garage Dining Table Amazon 03/03/2018 Classic F17YVQ8P 528.30 $',
     '169 Dining Room Tool Set Walmart 01/12/2018 Compact 2CH0K64T 1,247.92$',
     '170 Living Room Stand Mixer Target 29/08/2019 Compact HYMC4KDB 179.10 $',
     '171 Kitchen Tool Set Wayfair 24/10/2017 Premium A15U7SL6 752.20 $',
     '172 Office Mattress Wayfair 01/06/2015 Classic 631PVQZ8 954.68 $',
     '173 Living Room LED TV Amazon 06/09/2023 Compact LE7WASLW 1,227.77$',
     '174 Living Room Mattress Amazon 06/11/2016 Compact AE0U85VB 181.98 $',
     '175 Bedroom LED TV Best Buy 02/07/2019 Classic JMJI7NSB 417.10 $',
     '176 Dining Room Stand Mixer Best Buy 06/07/2016 Classic R1FEM8QE 601.05 $',
     '177 Bedroom LED TV Walmart 22/03/2019 Classic 83FL23MO 609.00 $',
     '178 Dining Room Tool Set Home Depot 04/05/2015 Compact X89Z8FCM 1,632.71$',
     '179 Kitchen LED TV Best Buy 15/11/2023 Premium MFRHLN1W 503.93 $        180 Garage LED TV Wayfair 19/12/2015 Premium I4N5J4VF 357.52 $',
     '181 Office Dining Table Walmart 07/12/2022 Classic 81XY8BYB 1,880.87$',
     '182 Dining Room Dining Table Wayfair 30/08/2016 Compact SFZM6UZB 1,240.03$',
     '183 Bedroom Dining Table Wayfair 02/06/2023 Modern QK2VYRQ4 1,164.27$',
     '184 Bedroom Dining Table Home Depot 10/07/2018 Compact MXCGC08A 325.25 $',
     '185 Kitchen Desk Best Buy 09/08/2018 Classic WM2DI61E 190.63 $',
     '186 Bedroom Dining Table Best Buy 26/04/2016 Classic UNFMT54I 1,176.83$',
     '187 Garage Mattress Walmart 09/09/2022 Compact 55GAE48V 670.44 $',
     '188 Bedroom Mattress Best Buy 26/04/2018 Modern R920ZMQD 1,474.32$',
     '189 Office Tool Set Target 23/03/2020 Compact LTPEQFTI 153.32 $',
     '190 Bedroom Desk Walmart 19/03/2024 Premium VB6ZKIQC 1,489.13$',
     '191 Bedroom LED TV Walmart 15/07/2018 Compact 1HI6TR6A 1,741.16$',
     '192 Garage Stand Mixer Walmart 16/04/2019 Premium DZCH7EWK 1,180.74$     Condition',
     'Excellent',
     'Excellent',
     'Excellent',
     'New',
     'Like New',
     'Like New',
     'Excellent',
     'Excellent',
     'Like New',
     'Like New',
     'Excellent',
     'Like New',
     'Excellent',
     'Good',
     'New',
     'Excellent',
     'Excellent',
     'New',
     'Like New',
     'Excellent',
     'Good',
     'Excellent',
     'Good',
     'Excellent',
     'Excellent',
     'New',
     'New',
     'Excellent',
     'Like New',
     'Like New',
     'New',
     'New',
     'New',
     'New',
     'New',
     'New',
     'Excellent',
     'ExcellentHOME INVENTORY Like New',
     'New',
     'New',
     'Excellent',
     'Excellent',
     'Like New',
     'Excellent',
     'Like New',
     'Excellent',
     'Excellent',
     'Good',
     'Excellent',
     'New',
     'Good',
     'Good',
     'New',
     'Excellent',
     'Excellent',
     'New',
     'Good',
     'Good',
     'Like New',
     'New',
     'Excellent',
     'Excellent',
     'Good',
     'Good',
     'Excellent',
     'Like New',
     'Excellent',
     'New',
     'Good',
     'Like New',
     'Like New',
     'Good',
     'New',
     'New',
     'Good',
     'Like New',
     'Good',
     'Excellent',
     'Good',
     'Like New',
     'Good',
     'Like New',
     'Excellent',
     'NewGood',
     'Like New',
     'Good',
     'Good',
     'Like New',
     'Like New',
     'Like New',
     'Like New',
     'Excellent',
     'Good',
     'Like New',
     'Excellent',
     'Good',
     'Excellent',
     'Like New',
     'Good',
     'New',
     'Excellent',
     'Good',
     'Good',
     'Good',
     'New',
     'Like New',
     'Good',
     'Excellent',
     'New',
     'New',
     'Like New',
     'New',
     'Excellent',
     'Good',
     'New',
     'Excellent',
     'Good',
     'Like New',
     'Like New',
     'Like New',
     'Like New',
     'Good',
     'New',
     'Good',
     'Like New',
     'Good',
     'New',
     'New',
     'Like New',
     'Like NewLike New',
     'New',
     'Excellent',
     'Excellent',
     'Like New',
     'Like New',
     'Good',
     'Like New',
     'Good',
     'Excellent',
     'Excellent',
     'Good',
     'Like New',
     'Good',
     'Good',
     'Good',
     'New',
     'New',
     'Like New',
     'Like New',
     'Like New',
     'Good',
     'Good',
     'Good',
     'Good',
     'New',
     'New',
     'Like New',
     'Excellent',
     'New',
     'Like New',
     'Like New',
     'Good',
     'New',
     'Good',
     'Like New',
     'Good',
     'Like New',
     'Good',
     'Like New',
     'Like New',
     'New',
     'Excellent',
     'New',
     'Good',
     'Good',
     'ExcellentNew',
     'Excellent',
     'New',
     'Good',
     'Good',
     'Good',
     'Like New',
     'New',
     'New',
     'Excellent',
     'Good',
     'New',
     'Excellent']




```python
def parse_date(date_str):
    """Convert date from metedata i extracted earlier in my code 'D:20250425205051Z' format to ISO format."""
    try:
        # Handle PDF metadata date format (D:YYYYMMDD...)
        if date_str.startswith('D:'):
            date_str = date_str[2:]
            
            year = date_str[0:4]
            month = date_str[4:6]
            day = date_str[6:8]
            
            hour = date_str[8:10] if len(date_str) > 8 else "00"
            minute = date_str[10:12] if len(date_str) > 10 else "00"
            second = date_str[12:14] if len(date_str) > 12 else "00"
            
            date_obj = datetime(int(year), int(month), int(day), 
                               int(hour), int(minute), int(second))
        
        # Handle DD/MM/YYYY format
        elif '/' in date_str:
            day, month, year = date_str.split('/')
            date_obj = datetime(int(year), int(month), int(day))
        
        else:
            raise ValueError(f"Unsupported date format: {date_str}")
            
        return date_obj.strftime("%Y-%m-%dT%H:%M:%S")
    except (ValueError, IndexError) as e:
        # In case the date is invalid
        print(f"Warning: Invalid date format - {date_str}. Error: {e}")
        return ""
date_str = 'D:20250425205051Z'
print(parse_date(date_str))
```

    2025-04-25T20:50:51
    


```python
parse_date(date_str)
```




    '2025-04-25T20:50:51'



## Getting to create the Full Pipeline Function

thios function runs the full pipeline and also saves the result to a JSON file.


```python

raw_text = get_data_from_pdf(pdf_path)


aligned_content = align_content(raw_text)


owner_info = extract_owner_info(aligned_content)
```


```python
def extract_owner_info(aligned_content):
    """Extract owner information from the content."""
    owner_info = OwnerInfo()
    
    for i, line in enumerate(aligned_content):
        if "Owner Information" in line:
            # Found the owner info section
            if i + 1 < len(aligned_content):
                owner_info.owner_name = aligned_content[i + 1]
            if i + 2 < len(aligned_content):
                owner_info.owner_address = aligned_content[i + 2]
            # Look for phone number
            for j in range(i+3, min(i+6, len(aligned_content))):
                if "(" in aligned_content[j] and ")" in aligned_content[j]:
                    owner_info.owner_telephone = aligned_content[j]
                    break
            break
    
    return owner_info
extract_owner_info(aligned_content)
```




    <__main__.OwnerInfo at 0x23c8164bd90>




```python
def find_inventory_table_start(aligned_content):
    """Find the starting index of the inventory table."""
    for i, line in enumerate(aligned_content):
        if "S/N Area" in line and "Item Description" in line:
            return i + 1  # Start from the next line
    
    print("Warning: Could not find inventory table header!")
    return -1

```


```python
import re
from datetime import datetime
#  Define function to parse inventory rows

def parse_inventory_row(line):
    """Parse a single inventory row and return an Inventory object."""
    # Check if line starts with a number (inventory item number)
    match = re.match(r'^(\d+)\s', line)
    if not match:
        return None  # Not an inventory row
    
    parts = line.split()
    
    # Need at least a minimum number of parts
    if len(parts) < 7:  # Number, Area, Description, Source, Date, Style, Serial, Value
        return None
    
    # First extract index & area
    idx = 1  # Start after the item number
    
    # Area might be one or two words like "Living Room"
    if idx + 1 < len(parts) and parts[idx+1].lower() == "room":
        area = f"{parts[idx]} {parts[idx+1]}"
        idx += 2
    else:
        area = parts[idx]
        idx += 1
    
    # Look for the date which has a specific format DD/MM/YYYY
    date_idx = -1
    for j, part in enumerate(parts):
        if re.match(r'\d{2}/\d{2}/\d{4}', part):
            date_idx = j
            break
    
    if date_idx == -1:
        # No date found, skip this row
        return None
    # Now we can work backwards and forwards from the date
    purchase_date = parts[date_idx]
    source = parts[date_idx - 1]
    style = parts[date_idx + 1]
    serial_number = parts[date_idx + 2]
    value = parts[-1].replace('$', '').replace(',', '')
    
    # Description is between area and source
    description_parts = parts[idx:date_idx-1]
    description = ' '.join(description_parts)
    
    # Create the source_style_area field
    source_style_area = f"{source} {style} {area}"
    
    # Format the date properly
    iso_date = parse_date(purchase_date)
    
    return Inventory(
        purchase_date=iso_date,
        serial_number=serial_number,
        description=description,
        source_style_area=source_style_area,
        value=value
    )
```


```python
# Define function to extract all inventory items
def extract_inventory_items(aligned_content):
    """Extract all inventory items from the content."""
    inventory_items = []
    table_start = find_inventory_table_start(aligned_content)
    
    if table_start == -1:
        return []
    
    # Process each inventory item row
    for i in range(table_start, len(aligned_content)):
        try:
            item = parse_inventory_row(aligned_content[i])
            if item:
                inventory_items.append(item.to_dict())
        except Exception as e:
            print(f"Error parsing line {i}: {e}")
            continue
    
    return inventory_items

```


```python
# Define the main extract_data function
def extract_data(aligned_content):
    """Extract structured data from aligned content."""
    owner_info = extract_owner_info(aligned_content)
    inventory_items = extract_inventory_items(aligned_content)
    
    # Create the final result dictionary
    result = {
        **owner_info.to_dict(),
        "data": inventory_items
    }
    
    return result
```


```python
#  Process the PDF and extract data
# I define the PDF path
pdf_path = "home_inventory.pdf" 

# Get raw text from PDF
raw_text = get_data_from_pdf(pdf_path)

# Clean and align the content
aligned_content = align_content(raw_text)

# Extract structured data
result = extract_data(aligned_content)

# Print the result in a readable format
print(json.dumps(result, indent=2))

# I also save to a JSON file
with open("extracted_inventory.json", "w") as f:
    json.dump(result, f, indent=2)
```

    {
      "owner_name": "John Doe",
      "owner_address": "123 Maple StreetName",
      "owner_telephone": "",
      "data": [
        {
          "purchase_date": "2021-04-22T00:00:00",
          "serial_number": "U7PXVQAB",
          "description": "Stand Mixer Best",
          "source_style_area": "Buy Modern Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2022-07-04T00:00:00",
          "serial_number": "4OGYNJGI",
          "description": "Tool Set Home",
          "source_style_area": "Depot Modern Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2016-05-11T00:00:00",
          "serial_number": "Z9TTB4MF",
          "description": "Desk Home",
          "source_style_area": "Depot Premium Garage",
          "value": ""
        },
        {
          "purchase_date": "2017-10-21T00:00:00",
          "serial_number": "7XW6R69R",
          "description": "Dining Table Home",
          "source_style_area": "Depot Classic Living Room",
          "value": ""
        },
        {
          "purchase_date": "2017-12-08T00:00:00",
          "serial_number": "CW9LEOK5",
          "description": "Desk",
          "source_style_area": "Amazon Classic Garage",
          "value": ""
        },
        {
          "purchase_date": "2023-05-17T00:00:00",
          "serial_number": "NBS18P9F",
          "description": "Desk Best",
          "source_style_area": "Buy Classic Office",
          "value": ""
        },
        {
          "purchase_date": "2021-06-07T00:00:00",
          "serial_number": "BYWR0VQ3",
          "description": "Dining Table",
          "source_style_area": "Wayfair Premium Kitchen",
          "value": "1331.28"
        },
        {
          "purchase_date": "2017-07-24T00:00:00",
          "serial_number": "Z4WKQAUD",
          "description": "Tool Set Home",
          "source_style_area": "Depot Compact Kitchen",
          "value": "1726.77"
        },
        {
          "purchase_date": "2020-06-11T00:00:00",
          "serial_number": "QRYP4WVD",
          "description": "Tool Set",
          "source_style_area": "Walmart Modern Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2016-08-10T00:00:00",
          "serial_number": "5DTN8WP1",
          "description": "Tool Set",
          "source_style_area": "Amazon Modern Dining Room",
          "value": "1364.42"
        },
        {
          "purchase_date": "2015-03-07T00:00:00",
          "serial_number": "S1IZ2ORO",
          "description": "Desk Best",
          "source_style_area": "Buy Compact Kitchen",
          "value": "1474.74"
        },
        {
          "purchase_date": "2022-10-07T00:00:00",
          "serial_number": "MSEPKFQG",
          "description": "Tool Set",
          "source_style_area": "Target Premium Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2016-09-28T00:00:00",
          "serial_number": "V5T93UBK",
          "description": "Mattress",
          "source_style_area": "Target Classic Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2023-04-29T00:00:00",
          "serial_number": "AE0EWUCB",
          "description": "Dining Table Home",
          "source_style_area": "Depot Compact Garage",
          "value": "1348.12"
        },
        {
          "purchase_date": "2021-10-28T00:00:00",
          "serial_number": "RESWYF9C",
          "description": "Dining Table",
          "source_style_area": "Target Compact Office",
          "value": ""
        },
        {
          "purchase_date": "2015-02-12T00:00:00",
          "serial_number": "GDKBSREF",
          "description": "Tool Set",
          "source_style_area": "Target Modern Office",
          "value": ""
        },
        {
          "purchase_date": "2018-11-05T00:00:00",
          "serial_number": "4R596I04",
          "description": "LED TV",
          "source_style_area": "Wayfair Compact Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2019-07-25T00:00:00",
          "serial_number": "PU3JA7WI",
          "description": "Dining Table",
          "source_style_area": "Target Modern Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2020-02-10T00:00:00",
          "serial_number": "CRUF4HVJ",
          "description": "Mattress",
          "source_style_area": "Target Premium Bedroom",
          "value": "1564.71"
        },
        {
          "purchase_date": "2020-08-18T00:00:00",
          "serial_number": "0N0SJGCY",
          "description": "Mattress",
          "source_style_area": "Amazon Compact Living Room",
          "value": "1384.72"
        },
        {
          "purchase_date": "2023-09-16T00:00:00",
          "serial_number": "V9W3YPMT",
          "description": "Dining Table",
          "source_style_area": "Amazon Classic Dining Room",
          "value": "1151.00"
        },
        {
          "purchase_date": "2023-05-31T00:00:00",
          "serial_number": "ORNZN8KE",
          "description": "Stand Mixer Best",
          "source_style_area": "Buy Premium Bedroom",
          "value": "1109.99"
        },
        {
          "purchase_date": "2019-09-12T00:00:00",
          "serial_number": "UR9LN2RL",
          "description": "Mattress Home",
          "source_style_area": "Depot Compact Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2025-09-24T00:00:00",
          "serial_number": "IYGUXBJ8",
          "description": "Stand Mixer",
          "source_style_area": "Wayfair Compact Garage",
          "value": ""
        },
        {
          "purchase_date": "2016-10-16T00:00:00",
          "serial_number": "R0S5K5A7",
          "description": "Tool Set",
          "source_style_area": "Wayfair Classic Office",
          "value": ""
        },
        {
          "purchase_date": "2025-09-11T00:00:00",
          "serial_number": "HJ62DG5N",
          "description": "Stand Mixer",
          "source_style_area": "Wayfair Premium Living Room",
          "value": ""
        },
        {
          "purchase_date": "2021-10-27T00:00:00",
          "serial_number": "9WBR8ESA",
          "description": "Tool Set Best",
          "source_style_area": "Buy Classic Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2025-12-15T00:00:00",
          "serial_number": "O4QZM9OP",
          "description": "Stand Mixer",
          "source_style_area": "Target Classic Office",
          "value": "1656.82"
        },
        {
          "purchase_date": "2019-02-08T00:00:00",
          "serial_number": "KJQP8H26",
          "description": "Desk",
          "source_style_area": "Amazon Premium Living Room",
          "value": "1386.27"
        },
        {
          "purchase_date": "2015-06-02T00:00:00",
          "serial_number": "U8DVZFWV",
          "description": "Mattress",
          "source_style_area": "Target Compact Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2024-04-13T00:00:00",
          "serial_number": "BJR8OZOF",
          "description": "Tool Set",
          "source_style_area": "Wayfair Modern Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2017-12-19T00:00:00",
          "serial_number": "DENRQBQ6",
          "description": "LED TV Home",
          "source_style_area": "Depot Premium Office",
          "value": "1219.77"
        },
        {
          "purchase_date": "2015-07-13T00:00:00",
          "serial_number": "2CZTWBEN",
          "description": "Dining Table",
          "source_style_area": "Walmart Modern Office",
          "value": "1676.31"
        },
        {
          "purchase_date": "2020-05-21T00:00:00",
          "serial_number": "ELLVSI6B",
          "description": "LED TV",
          "source_style_area": "Wayfair Premium Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2023-03-20T00:00:00",
          "serial_number": "WWMYJEAV",
          "description": "Mattress",
          "source_style_area": "Walmart Classic Office",
          "value": ""
        },
        {
          "purchase_date": "2021-04-28T00:00:00",
          "serial_number": "E8LNBEOM",
          "description": "Tool Set",
          "source_style_area": "Wayfair Modern Garage",
          "value": ""
        },
        {
          "purchase_date": "2017-04-22T00:00:00",
          "serial_number": "DCATHJ3H",
          "description": "Tool Set",
          "source_style_area": "Wayfair Premium Living Room",
          "value": "1576.72"
        },
        {
          "purchase_date": "2021-05-31T00:00:00",
          "serial_number": "0VA5WHK9",
          "description": "LED TV",
          "source_style_area": "Amazon Classic Kitchen",
          "value": "1044.09"
        },
        {
          "purchase_date": "2022-10-09T00:00:00",
          "serial_number": "JLKFBBS5",
          "description": "LED TV",
          "source_style_area": "Amazon Premium Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2018-12-13T00:00:00",
          "serial_number": "UW76JUEY",
          "description": "Tool Set",
          "source_style_area": "Wayfair Compact Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2021-11-29T00:00:00",
          "serial_number": "WEVV8O3R",
          "description": "LED TV",
          "source_style_area": "Wayfair Compact Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2020-11-23T00:00:00",
          "serial_number": "ENVSPMO9",
          "description": "Dining Table",
          "source_style_area": "Amazon Premium Living Room",
          "value": ""
        },
        {
          "purchase_date": "2018-12-04T00:00:00",
          "serial_number": "MZ2LZRV5",
          "description": "Stand Mixer Best",
          "source_style_area": "Buy Compact Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2017-10-27T00:00:00",
          "serial_number": "7K0G5I95",
          "description": "LED TV Best",
          "source_style_area": "Buy Modern Bedroom",
          "value": "1677.74"
        },
        {
          "purchase_date": "2022-01-23T00:00:00",
          "serial_number": "MO66OFGP",
          "description": "Desk",
          "source_style_area": "Walmart Compact Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2017-02-10T00:00:00",
          "serial_number": "X8RLUG3J",
          "description": "Tool Set",
          "source_style_area": "Target Premium Garage",
          "value": "1438.51"
        },
        {
          "purchase_date": "2021-09-21T00:00:00",
          "serial_number": "55DBVC6I",
          "description": "Tool Set Best",
          "source_style_area": "Buy Modern Bedroom",
          "value": "1464.33"
        },
        {
          "purchase_date": "2021-03-04T00:00:00",
          "serial_number": "BQ2K5ZP6",
          "description": "LED TV",
          "source_style_area": "Walmart Premium Office",
          "value": ""
        },
        {
          "purchase_date": "2017-01-04T00:00:00",
          "serial_number": "9OX4W1DP",
          "description": "LED TV Home",
          "source_style_area": "Depot Compact Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2017-08-04T00:00:00",
          "serial_number": "08RQPP9E",
          "description": "Dining Table Home",
          "source_style_area": "Depot Classic Dining Room",
          "value": "1513.78"
        },
        {
          "purchase_date": "2018-10-22T00:00:00",
          "serial_number": "BHQOC2AK",
          "description": "LED TV Best",
          "source_style_area": "Buy Compact Office",
          "value": "1156.85"
        },
        {
          "purchase_date": "2022-10-15T00:00:00",
          "serial_number": "27QA3NCY",
          "description": "Stand Mixer",
          "source_style_area": "Amazon Modern Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2017-04-20T00:00:00",
          "serial_number": "828ZXE71",
          "description": "Stand Mixer",
          "source_style_area": "Amazon Classic Dining Room",
          "value": "1129.03"
        },
        {
          "purchase_date": "2022-01-02T00:00:00",
          "serial_number": "K0M9NKF5",
          "description": "Dining Table Home",
          "source_style_area": "Depot Classic Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2019-07-13T00:00:00",
          "serial_number": "Y2O7L2B0",
          "description": "LED TV",
          "source_style_area": "Walmart Compact Kitchen",
          "value": "1394.44"
        },
        {
          "purchase_date": "2024-02-07T00:00:00",
          "serial_number": "NPS1GVZY",
          "description": "Desk",
          "source_style_area": "Wayfair Modern Kitchen",
          "value": "1484.40"
        },
        {
          "purchase_date": "2020-06-12T00:00:00",
          "serial_number": "E6DYBIWE",
          "description": "LED TV",
          "source_style_area": "Amazon Modern Dining Room",
          "value": "1461.72"
        },
        {
          "purchase_date": "2021-01-21T00:00:00",
          "serial_number": "ALOGE3QY",
          "description": "LED TV",
          "source_style_area": "Walmart Compact Garage",
          "value": ""
        },
        {
          "purchase_date": "2021-12-11T00:00:00",
          "serial_number": "LBS7UPM6",
          "description": "LED TV Best",
          "source_style_area": "Buy Modern Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2019-12-06T00:00:00",
          "serial_number": "R5YCRXSQ",
          "description": "LED TV",
          "source_style_area": "Wayfair Compact Office",
          "value": "1273.78"
        },
        {
          "purchase_date": "2019-04-13T00:00:00",
          "serial_number": "NQKV79JA",
          "description": "Tool Set",
          "source_style_area": "Target Premium Living Room",
          "value": "1750.27"
        },
        {
          "purchase_date": "2015-05-13T00:00:00",
          "serial_number": "P0ZY0J5A",
          "description": "LED TV Best",
          "source_style_area": "Buy Compact Garage",
          "value": ""
        },
        {
          "purchase_date": "2025-03-15T00:00:00",
          "serial_number": "U0EGB2H1",
          "description": "LED TV Best",
          "source_style_area": "Buy Premium Office",
          "value": "1624.11"
        },
        {
          "purchase_date": "2017-03-20T00:00:00",
          "serial_number": "T96GB8RE",
          "description": "Tool Set",
          "source_style_area": "Walmart Premium Living Room",
          "value": "1873.35"
        },
        {
          "purchase_date": "2017-09-28T00:00:00",
          "serial_number": "IZ10KLG2",
          "description": "Desk",
          "source_style_area": "Target Premium Kitchen",
          "value": "1286.29"
        },
        {
          "purchase_date": "2020-10-24T00:00:00",
          "serial_number": "EG70TSWM",
          "description": "Desk",
          "source_style_area": "Wayfair Classic Dining Room",
          "value": "1747.33"
        },
        {
          "purchase_date": "2020-01-15T00:00:00",
          "serial_number": "HUGYL1UA",
          "description": "LED TV",
          "source_style_area": "Walmart Compact Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2018-03-03T00:00:00",
          "serial_number": "F17YVQ8P",
          "description": "Dining Table",
          "source_style_area": "Amazon Classic Garage",
          "value": ""
        },
        {
          "purchase_date": "2018-12-01T00:00:00",
          "serial_number": "2CH0K64T",
          "description": "Tool Set",
          "source_style_area": "Walmart Compact Dining Room",
          "value": "1247.92"
        },
        {
          "purchase_date": "2019-08-29T00:00:00",
          "serial_number": "HYMC4KDB",
          "description": "Stand Mixer",
          "source_style_area": "Target Compact Living Room",
          "value": ""
        },
        {
          "purchase_date": "2017-10-24T00:00:00",
          "serial_number": "A15U7SL6",
          "description": "Tool Set",
          "source_style_area": "Wayfair Premium Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2015-06-01T00:00:00",
          "serial_number": "631PVQZ8",
          "description": "Mattress",
          "source_style_area": "Wayfair Classic Office",
          "value": ""
        },
        {
          "purchase_date": "2023-09-06T00:00:00",
          "serial_number": "LE7WASLW",
          "description": "LED TV",
          "source_style_area": "Amazon Compact Living Room",
          "value": "1227.77"
        },
        {
          "purchase_date": "2016-11-06T00:00:00",
          "serial_number": "AE0U85VB",
          "description": "Mattress",
          "source_style_area": "Amazon Compact Living Room",
          "value": ""
        },
        {
          "purchase_date": "2019-07-02T00:00:00",
          "serial_number": "JMJI7NSB",
          "description": "LED TV Best",
          "source_style_area": "Buy Classic Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2016-07-06T00:00:00",
          "serial_number": "R1FEM8QE",
          "description": "Stand Mixer Best",
          "source_style_area": "Buy Classic Dining Room",
          "value": ""
        },
        {
          "purchase_date": "2019-03-22T00:00:00",
          "serial_number": "83FL23MO",
          "description": "LED TV",
          "source_style_area": "Walmart Classic Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2015-05-04T00:00:00",
          "serial_number": "X89Z8FCM",
          "description": "Tool Set Home",
          "source_style_area": "Depot Compact Dining Room",
          "value": "1632.71"
        },
        {
          "purchase_date": "2023-11-15T00:00:00",
          "serial_number": "MFRHLN1W",
          "description": "LED TV Best",
          "source_style_area": "Buy Premium Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2022-12-07T00:00:00",
          "serial_number": "81XY8BYB",
          "description": "Dining Table",
          "source_style_area": "Walmart Classic Office",
          "value": "1880.87"
        },
        {
          "purchase_date": "2016-08-30T00:00:00",
          "serial_number": "SFZM6UZB",
          "description": "Dining Table",
          "source_style_area": "Wayfair Compact Dining Room",
          "value": "1240.03"
        },
        {
          "purchase_date": "2023-06-02T00:00:00",
          "serial_number": "QK2VYRQ4",
          "description": "Dining Table",
          "source_style_area": "Wayfair Modern Bedroom",
          "value": "1164.27"
        },
        {
          "purchase_date": "2018-07-10T00:00:00",
          "serial_number": "MXCGC08A",
          "description": "Dining Table Home",
          "source_style_area": "Depot Compact Bedroom",
          "value": ""
        },
        {
          "purchase_date": "2018-08-09T00:00:00",
          "serial_number": "WM2DI61E",
          "description": "Desk Best",
          "source_style_area": "Buy Classic Kitchen",
          "value": ""
        },
        {
          "purchase_date": "2016-04-26T00:00:00",
          "serial_number": "UNFMT54I",
          "description": "Dining Table Best",
          "source_style_area": "Buy Classic Bedroom",
          "value": "1176.83"
        },
        {
          "purchase_date": "2022-09-09T00:00:00",
          "serial_number": "55GAE48V",
          "description": "Mattress",
          "source_style_area": "Walmart Compact Garage",
          "value": ""
        },
        {
          "purchase_date": "2018-04-26T00:00:00",
          "serial_number": "R920ZMQD",
          "description": "Mattress Best",
          "source_style_area": "Buy Modern Bedroom",
          "value": "1474.32"
        },
        {
          "purchase_date": "2020-03-23T00:00:00",
          "serial_number": "LTPEQFTI",
          "description": "Tool Set",
          "source_style_area": "Target Compact Office",
          "value": ""
        },
        {
          "purchase_date": "2024-03-19T00:00:00",
          "serial_number": "VB6ZKIQC",
          "description": "Desk",
          "source_style_area": "Walmart Premium Bedroom",
          "value": "1489.13"
        },
        {
          "purchase_date": "2018-07-15T00:00:00",
          "serial_number": "1HI6TR6A",
          "description": "LED TV",
          "source_style_area": "Walmart Compact Bedroom",
          "value": "1741.16"
        },
        {
          "purchase_date": "2019-04-16T00:00:00",
          "serial_number": "DZCH7EWK",
          "description": "Stand Mixer",
          "source_style_area": "Walmart Premium Garage",
          "value": "Condition"
        }
      ]
    }
    


```python

```

## Challenges and Notes

While implementing this, I faced a few challenges:

1. PDF parsing can be messy - the text alignment might not be exactly as it appears visually in the PDF
2. The inventory items have variable formats, making it tricky to parse consistently
3. Date formatting needed special handling to convert to ISO format

Some improvements I could make with more time:
- Better error handling for edge cases
- More robust parsing of complex inventory descriptions
- Add unit tests to verify each function works correctly
