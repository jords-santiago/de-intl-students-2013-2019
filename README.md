# Data Engineering Project for International Students 2013-2019 Data Analysis

## Introduction
### Background

This contains and details how data regarding International Students Enrollment from 2013 to 2019 has been extracted, transformed then loaded to a database for further use/analysis using AWS services.

For the actual analysis (not using cloud services), you may refer to [this](https://github.com/jords-santiago/intl-students-2013-2019).

### Objective

* Extract, transform and load data using cloud resources (AWS)

## Methodology
### Data Sources

#### International Student Enrollment

For the International Students data, this can be acquired from the Organisation for Economic Cooperation and Development (OECD) online library.  This has an online Education Statistics database gathered from educational institutions.  The dataset specifically acquired from this library is the [Enrollment of international students by origin](https://stats.oecd.org/viewhtml.aspx?datasetcode=EDU_ENRL_MOBILE&lang=en).  This dataset mainly contains international student enrollment for post-secondary education (tertiary level or higher).  The dataset mainly has enrollment data from 2013 to 2019 **only for OECD countries**.  OECD does have enrollment data for international students going to non-OECD countries but those were not categorized for a given calendar year.  For the purpose of this project, we can take data on OECD countries as those are where the bulk of international students enroll to.

The [full dataset in CSV format](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/01_DataSource/01_Raw/01_OECD/EDU_ENRL_MOBILE-en.csv.zip) was downloaded as shown below.

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/OECD_data_download.png "Downloading OECD dataset")  

#### World Development Indicators

For the World Development Indicators data, this can be found in the World Bank Open Data site.  From its [World Development Indicators DataBank](https://databank.worldbank.org/source/world-development-indicators#), specified indicators can be queried for a given set of countries across a period of time as shown below.  Since the OECD data covers years 2013 to 2019, the [data extract in CSV format](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/01_DataSource/01_Raw/02_WORLD_BANK/Data_Extract_From_World_Development_Indicators.zip) from this databank covers that time period as well.

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/World_Bank_data_download.png "Downloading World Bank dataset") 

The indicators gathered were the following:

| Indicator Code | Description |
| --- | --- |
| NY.GDP.MKTP.KD.ZG	| GDP growth (annual %) |
| NY.GDP.PCAP.CD| GDP per capita (current US$) |
| NY.GDP.MKTP.CD | GDP (current US$) |
| SP.POP.TOTL | Population, total |
| SE.XPD.CTER.ZS | Current education expenditure, tertiary (% of total expenditure in tertiary public institutions) |
| SE.XPD.TERT.PC.ZS	| Government expenditure per student, tertiary (% of GDP per capita) |
| SE.XPD.TERT.ZS | Expenditure on tertiary education (% of government expenditure on education) |
| EN.POP.DNST | Population density (people per sq. km of land area) |
| SL.UEM.ADVN.ZS | Unemployment with advanced education (% of total labor force with advanced education) |
| SP.POP.GROW | Population growth (annual %) |
| SP.URB.GROW | Urban population growth (annual %) |
| NY.GDP.PCAP.KD.ZG	| GDP per capita growth (annual %) |
| SP.RUR.TOTL.ZG | Rural population growth (annual %) |

The data from the World Bank needed to be arranged where World Development Indicators per year had its own column.  Unpivot was performed using **Google Sheets** to have the year specified as a row.  The transformed dataset was extracted as a new [CSV file](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/01_DataSource/01_Raw/02_WORLD_BANK/WORLD_BANK_SELECTED_WDI_2013_2019.zip) to be loaded with the other 2 datasets.

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/World_Bank_WDI_prep.png "Unpivot the World Bank dataset") 

#### ISO 3166 Country Codes

Upon checking the datasets for International Students and World Development Indicators, country codes were also given.  These were 2-character (alpha-2) or 3-character (alpha-3) codes for the countries in compliance of [ISO 3166](https://www.iso.org/iso-3166-country-codes.html).  However, the country names themselves don't match between the 2 datasets.  Therefore, a [CSV file](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/01_DataSource/01_Raw/03_ISO3166/ISO_3166_COUNTRY_CODES.csv) was created for the ISO 3166 country codes to be able to join the datasets for further analysis.

### Data Engineering Flow

The figure below details the flow from raw data from the previous section to their final form loaded into the database.

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/de-flow-intl-students.png "Data Flow") 

The 3 datasets (in csv format) were uploaded into **AWS S3 Storage**.

* EDU_ENRL_MOBILE-en.csv
* WORLD_BANK_SELECTED_WDI_2013_2019.csv
* ISO_3166_COUNTRY_CODES.csv

To extract the data, **AWS Glue Crawler** was used to load data into **AWS Glue Data Catalog** into separate tables described below:

| CSV file name | Data Catalog Database | Table Name |
|-------------|-----------|-----------|
| EDU_ENRL_MOBILE-en.csv | intl_students_raw | oecd |
| WORLD_BANK_SELECTED_WDI_2013_2019.csv | intl_students_raw | worldbank |
| ISO_3166_COUNTRY_CODES.csv | intl_students_raw | iso3166 | 

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/aws_crawlers.png "AWS Crawlers")

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/aws_glue_data_catalog_via_athena.png "AWS Glue Data Catalog") 

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/aws_glue_data_catalog_table.png "intl_students_raw Tables in Glue Data Catalog") 

Once these were loaded into the data catalog, data would be transformed and loaded into **AWS Redshift** using **AWS Glue ETL**.  The script/code can be found [here](https://github.com/jords-santiago/intl-students-2013-2019/blob/main/02_SourceCodes/ETLjob.ipynb).

![alt text](https://github.com/jords-santiago/de-intl-students-2013-2019/blob/main/99_Pictures/aws_glue_etl.png "Transformation using AWS Glue ETL")  

In summary, the following transformations were performed:

* Filtered out data to only include total number of students (excluded numbers separating males and females)
* Filtered out invalid years (9999 was set as a catch-all time period) as well as outside of 2013-2019 period
* Rounded down/truncated values as number of students should be specified as whole numbers
* Converted country codes into actual country names (using ISO 3166 country codes dataset)
* Remove redundant values regarding Education Level
* Joined Population data from World Bank dataset into International Students data

Using Glue ETL has yielded 3 output tables:

| Output Table Name | Output CSV File | Description |
| --- | --- |
| OECD_EDU_ENRL_2013_2019 | International Students' Enrollment 2013-2019 | 
| INTL_STUDENTS_PER_POPULATION | International Students' Enrollment 2013-2019 with Population |
| INTL_STUDENT_ORIGIN_2013_2019 | International Students' Countries of Origin 2013-2019 |

These table should now be ready for use for further analysis/visualization.

## Key Takeaways

Having able to perform this project using "on-premises" resources but on different platforms (MS SQL and then Python/Jupyter Notebook), it is great that there is an option to have these all provided using AWS services.

This project can further be expanded by having scheduled jobs instead of manual triggers.  Also, extraction of source data can then be done using APIs instead of manual download for a more automated data flow.

However, this project is able to show an application of typical data engineering work (ETL) via cloud resources.
