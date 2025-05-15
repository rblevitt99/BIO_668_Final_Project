# BIO_668_Final_Project
Here is some code I used in python and R to identify differentially expressed transporter genes in a transcriptomics data set


import pandas as pd
#from openpyxl import load_workbook
#from openpyxl import Workbook
import openpyxl
import xlsxwriter


# This first function will take the Comparisons transcriptomics data set and find all the differentially expressed (DE)
# genes. The DE genes will be found by filtering for any gene that has either a log2 fold change 
# > 1 or <-1 and has a p-value of < 0.05.



workbook=openpyxl.Workbook()
workbook.save(filename="Filtered_DE_Gene_data.xlsx")

sheet=["LowCa_vs_Ca","Ca_vs_Low_Fe", "LowFe_vs_Fe","Fe_vs_Mix","Mix_vs_Ni",
"Ni_vs_Nd","Nd_vs_W","W_vs_Cu"] # define the sheets within in the log2 comparisons data frame and the sheets that will exist in the output data frame

def DE_Conductor(input):
    df=pd.read_excel("Transcriptomics_Comparisons_Data.xlsx",sheet_name=input) # using pandas and read_excel to read in my comparisons dataframe. Notice that I am also added "sheet_name" to specify which sheet I want to work with. This is the part that will change multiple time. Thus why I am writing a fuction...
    filtered_data = df[((df['log2FoldChange'] > 1) | (df['log2FoldChange'] < -1)) & (df['pvalue'] < 0.05)] # This code filters through the excel file and only picks out genes that have a log2 fold change above 1 or below -1 and have a p-value that is less than 0.05
    with pd.ExcelWriter( "Filtered_DE_Gene_data.xlsx", mode="a", engine="openpyxl", if_sheet_exists="new") as writer: # This code is used to 1. open and close the file with "with", 2. create a new excel object which will be "Filtered_DE_Gene_data.xlsx". This is the new excel file I want to fill up the differentially expressed data with. 3. mode="a" means I want append mode, as I want to append the DE values to this excel file. 
        filtered_data.to_excel(writer, sheet_name=input, index=False) # writing the information stored in the variable "filtered data" to this new excel file
for input in sheet: #This for loop is iterating through the above function so that I can filter for DE genes for every sheet.
    DE_Conductor(input)

for sheetname in sheet:
    print(len(pd.read_excel("Filtered_DE_Gene_data_Removed_Empty_refseq.xlsx",sheet_name=sheetname))) # This just prints the count of genes in each sheet to give me an idea of how many DE genes there are in each condition





#This next function will map the COG ID's to the DE gene data set that was just created.

sheet=["LowCa_vs_Ca","Ca_vs_Low_Fe", "LowFe_vs_Fe","Fe_vs_Mix","Mix_vs_Ni",
"Ni_vs_Nd","Nd_vs_W","W_vs_Cu"]


def COG_Mapper(input):
    deg_file = "Filtered_DE_Gene_data.xlsx"  # Setting file to def_file variable
    cog_file = "COG_ID's.xlsx" #Setting file to cog_file variable
    # Open the DE gene Excel file for writing. The mode='a' means append mode.
    # engine='openpyxl' allows working with Excel files.
    # if_sheet_exists='overlay' lets you overwrite the sheet if it already exists.
    with pd.ExcelWriter(deg_file, mode="a", engine="openpyxl", if_sheet_exists="overlay") as writer:
        deg_df = pd.read_excel(deg_file, sheet_name=input) # reading the right sheet in the deg_file. 
        cog_df = pd.read_excel(cog_file) # reading the cog data file
        merged_df = pd.merge(deg_df, cog_df, on="refseq_protein_id", how="left") # combining the two files together by merging on the 'refseq_protein_id'. Each file has the same 'refseq_protein_id. The deg file has refseq_protein_id's for each DE gene and the cog file has the same refseq_protein_ids for each protein in the dataframe. The cog dataframe has every gene in the 20z genome. Thus, I should get hits when merging these two together.
        merged_df["COG_category"].fillna("Unknown", inplace=True) # if there is any empty cell, just fill it with an NA
        merged_df.to_excel(writer, sheet_name=input, index=False) # write all this crud to the excel file. 
for input in sheet: # utilize this function multiple times so that the same process occurs for each sheet. 
    COG_Mapper(input)


# Now that we mapped the COG ID's to the DE gene data set we now want to reduce the size of this data set 
# and select specific columns that we care about. Those columns are: gene product, 
# Averaged Log2Ratio, p-value, refseq_locus,refseq id, COG category, and the description of the COG catergory associated with the 
# aligned gene.

# workbook=openpyxl.Workbook()
# workbook.save(filename="Transcriptomics_DE_COG_Desired_Columns.xlsx")

# sheet = ["LowCa_vs_Ca","Ca_vs_Low_Fe","LowFe_vs_Fe","Fe_vs_Mix", "Mix_vs_Ni", 
# "Ni_vs_Nd", "Nd_vs_W", "W_vs_Cu", "lowCa_vs_Cu"]


# def cog_sorter(input):
#     df = pd.read_excel("Filtered_DE_Gene_data.xlsx",sheet_name=input)
#     desired_columns=["product", "log2FoldChange", "pvalue", "ID","refseq_protein_id","COG_category", "Description_y"] # setting up parameters for which columns I want.
#     df_selected=df[desired_columns] # By doing df[desired_columns], I am selecting the columns listed in "desired_columns" from df (the filtered DE gene data frame)
#     path = "Transcriptomics_DE_COG_Desired_Columns.xlsx"
#     with pd.ExcelWriter(path, mode='a', engine='openpyxl',if_sheet_exists="new") as writer:
#         df_selected.to_excel(writer, sheet_name=input, index=False)
# for input in sheet:
#     cog_sorter(input)


# #Now that we have a smaller data set with the desired columns, lets now split up the excel file into
# # mutliple sheets where each sheet will detail which genes in each condition are mapped to a 
# # specific COG category we are interested in. In this case I am interested in any COG ID that is
# # associated with transporter function. If you look at the list called "Desired_COG" you will 
# # see which COG id's I am curious about. 

workbook=openpyxl.Workbook()
workbook.save(filename="Filtered_DE_Genes_COG_Transporters.xlsx")

filtered_COG_Genes=["LowCa_vs_Ca","Ca_vs_Low_Fe","LowFe_vs_Fe","Fe_vs_Mix", "Mix_vs_Ni", 
"Ni_vs_Nd", "Nd_vs_W", "W_vs_Cu"]

Desired_COG = ["H", "P", "U", "Q", "W"]

def COG_transporter_sorter(input_sheet):
    df=pd.read_excel("Transcriptomics_DE_COG_Desired_Columns.xlsx",sheet_name=input_sheet) # setting the in file to df
    path = "Filtered_DE_Genes_COG_Transporters.xlsx" # setting the outfile to path
    with pd.ExcelWriter(path,mode='a',engine='openpyxl', if_sheet_exists="new") as writer: # opening the output path file for writing
        for COG_id in Desired_COG: # This for loop iterates through the list "desired COG and if the COG category exists in the "Transcriptomics_DE_COG_Desired_Columns" dataframe it is appended to the current sheet.
            Filteted_Transporters = df[df['COG_category'] == COG_id]
            Filteted_Transporters.to_excel(writer, sheet_name=input_sheet, index=False)
for input_sheet in filtered_COG_Genes: # loop this function for each sheet.
     COG_transporter_sorter(input_sheet)
