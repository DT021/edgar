# EDGAR

 The Electronic Data Gathering, Analysis, and Retrieval system (EDGAR) is an online repository, first constructed in 1993, which is maintained by the US Securities and Exchange Commission (SEC), and is designed to automate the collection, validation and acceptance of submissions and announcements by companies that are required to do so by law. The edgar schema is a collection of information scraped from this repository. The tables contained in this database schema consist of a number of main tables, containing the most fundamental information contained in EDGAR, as well as a number of dependent tables which are constructed most generally by scraping and cleaning the information in the filings and documents linked from the main tables, such as the set of tables listing the information contained in the Form 3, Form 4, and Form 5 filings. This readme file discusses the main tables. By far the most important tables are `filings` and `filing_docs`, which contain the basic information on each filing and their linking documents respectively, which one can use to deduce the url links to get to them, or to find the location of the documents in memory if they have been downloaded.


## `filings` and `filing_docs`

As mentioned above, these are main tables indexing all the filings in the SEC EDGAR database since 1993, and their associated documents respectively. Here, for each table we give a list of the associated fields. 

* `filings`: This is an index of all filings in the SEC EDGAR database (since 1993). The program which constructs this table is contained in [`get_filings.R`](get_filings.R). The fields are

    - `company_name`: the name of the company/firm which made the filing.          
    - `form_type`: the form type of the filing. A description of the existing form types can be found [here](https://www.sec.gov/info/edgar/forms/edgform.pdf).
    - `cik`: the Central Index Key (CIK) of the company/firm which made the filing. This index is very important, as each company/firm is assigned a unique CIK number, and it can thus be used to efficiently search for the filings of a particular company/firm on EDGAR.   
    - `date_filed`: the date the filing was made with the SEC.
    - `file_name`: This is the file name of the filing. This is an important field, as the file name uniquely assigned for each filing, and so can therefore identify each filing uniquely. This makes it important for joining with dependant tables, as they are often indexed, either partially or fully, by this file name.     
    
* `filing_docs`: this is a table listing the documents associated with each filing that appears in `filings`. The code which maintains this table is contained in [`get_filing_docs.R`](get_filing_docs.R). See [here](filing_docs.md) for more information. The fields are    

    - `seq`: The sequence number, or ordinal number, of the document as it appears on the html page of the filing.
    - `description`: this is a description of what the document is.
    - `document`: this is a file name for the particular document. This, along with `file_name`, is also important for joining with dependant tables. The tuple of (`file_name`, `document`) uniquely identifies a particular document for a particular filing, and records from dependant tables are thus usually indexed by these fields (eg. the tables containing the data from Forms 3, 4 and 5).
    - `type`:  a word describing what type of document the given document is.
    - `size`:  the size of the file in bytes, as stated in the page of the filing
    - `file_name`: This is the same as in `filings`               


## Other central tables

* `accession_numbers`: Each filing has an accession number, contained in the `file_name` of the filing in `filings`. Each accession number contains three numbers: the first number is the CIK, the second is the year in which the filing was made, and the third is a sequential count of the number of filings made under the CIK (usually for the year, see [here](https://www.sec.gov/edgar/searchedgar/accessing-edgar-data.htm) for more detail). This table contains these. The code to make this table is in [`get_accession_nos.R`](get_accession_nos.R). The fields are

    - `file_name`: Same as in `filings`
    - `accessionnumber`: the accession number, represented as a string. 



* `cusip_cik`: CUSIP numbers are unique identifying numbers for North American securities, assigned by the Committee on Uniform Security Identification Procedures (CUSIP), hence the name. This table provides a mapping from CUSIPs to CIKs. The mapping is formed by scraping the CIK and CUSIP numbers for each company listed in forms `SC 13D` and `SC 13G`, and associating them. The fields are

    - `file_name`: Same as in `filings`. In this case, this is the file name of the filing from which the data in the other fields was scraped.            
    - `cusip`: the CUSIP number of the company.
    - `cik`: The CIK number of the company. This is not necessary the same CIK as that contained in the `file_name`.
    - `company_name`: The name of the company.
    - `format`: A letter (usually 'A', 'B' or 'D') which keeps track of how the CUSIP number is displayed in the original filing document. 




* `filer_ciks`: This table contains data on filer CIKs from `SC 13D` and `SC 13G` forms. The code which makes this table is contained in [`get_filer_ciks.R`](get_filer_ciks.R). The fields are

    - `file_name`: Same as in `filings`, this case the file name of the particular `SC 13D` or `SC 13G` form.
    - `filer_cik`: the CIK number of the filer




* `item_no`: Table listing item numbers associated with each 8-K filing. The code to make and update this table is contained in [`get_item_nos.R`](get_item_nos.R). The fields are

    - `file_name`: Same as in `filings`, this case the file name of the particular `8-K` or `8-K/A` form.
    - `item_no`: the item number associated with the filing. If a filings has more than one item number, there will be multiple rows for the filings associated with each item number. A description of these item numbers are given in the following table, `item_no_desc`.



* `item_no_desc`: Table providing explanations for each item number used in 8-K filings. The data is extracted from [here](https://www.sec.gov/fast-answers/answersform8khtm.html). The code to make the table is contained in [`get_item_no_desc.R`](get_item_no_desc.R), which scrapes the descriptions for each of the unique item numbers stored in `item_no`. The fields are

    - `item_no`: the item number. Same as the numbers that appear in the corresponding field in the table `item_no`.
    - `item_desc`: A description of the situation/context that the item number corresponds to.

- `server_log`: **Details to come.**

### Tables used in updating tables above

- `index_last_modified`: Table used in updating `filings`.
- `filing_docs_processed`: Table used in keeping track of documents in `filing_docs` that have been downloaded using [download_filing_docs.R](download_filing_docs.R).
- `server_log_processed`: Table used in updating `server_log`.

## Code

A script to update the Edgar database is found [here](update_edgar.sh).

- [`get_server_logs.R`](server_logs/get_server_logs.R): Script to get server logs from EDGAR to populate `server_log` and `server_log_processed`.
- [`get_13D_filings.R`](get_13D_filings.R): Script to get the 13D filings. Always run this before running `extract_cusips.R`.
- [`get_13D_filing_details`](get_13D_filing_details.R): Extracts the details from the files contained in `edgar.filings`
- [`extract_cusips.pl`](extract_cusips.pl): Contains Perl code which extracts the cusip_cik numbers. This file is used and sourced by extract_cusips.R, so one should run the latter to put the cusip_ciks in the database.
- [`extract_cusips.R`](extract_cusips.R): Puts the cusip_cik numbers in the database.
