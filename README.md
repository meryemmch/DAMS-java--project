# DAMS – Java Data Analysis & Reporting Project
 
A menu-driven Java console application that loads e-commerce CSV data (customers, products, orders), cleans it, computes descriptive statistics, and generates HTML analysis reports. Access is protected by a simple sign-up / log-in system.
 
## Features
 
- **User authentication** – sign up with validated name, email, phone number, username and password; log in against a CSV-backed user store.
- **Data cleaning pipeline** for each dataset:
  1. Read the CSV file
  2. Remove duplicates (by customer / product / order ID)
  3. Remove rows with missing or invalid values
  4. Draw a random sample of 200 records
  5. Run the analysis and print results to the console
  6. Save the processed sample to an output CSV
- **Analysis**
  - *Customers*: count by gender, count by age group, top 5 cities.
  - *Products*: categories and brands, product count per category, cheapest / most expensive product overall and per category.
  - *Orders*: total revenue, orders by payment method, average discount and shipping cost, rating and delivery-status frequencies, best/worst rating counts, highest-value order, max/min discount per product, average rating per product, most bought product, mode of quantity per product, monthly sales, and simple linear regressions of total amount against quantity, shipping cost and discount.
- **HTML reports** – one report per dataset, written to the project root.
## Project structure
 
```
DAMS-java--project/
├── resources/
│   ├── customers.csv        # raw customer data
│   ├── products.csv         # raw product data
│   ├── order.csv            # raw order data
│   ├── users.csv            # registered users (created/appended at sign-up)
│   ├── outputcustomer.csv   # processed customer sample (generated)
│   ├── outputproduct.csv    # processed product sample (generated)
│   └── outputorder.csv      # processed order sample (generated)
├── src/main/java/
│   ├── Main.java                         # entry point and menus
│   ├── dataprocessing/
│   │   ├── DataProcessor.java            # generic abstract base class
│   │   ├── DataProcessing.java           # cleaning interface
│   │   ├── DataReading.java              # reading interface
│   │   ├── DataStoring.java              # storing interface
│   │   ├── Analyzable.java               # analysis interface
│   │   ├── CustomerDataProcessor.java
│   │   ├── ProductDataProcessor.java
│   │   └── OrderDataProcessor.java
│   ├── model/                            # Person, User, Customer, product, Order
│   ├── ReportGenerator/
│   │   └── ReportGenerator.java          # builds the HTML reports
│   └── userManager/                      # UserManager and input validators
├── customer_data_analysis_report.html    # generated report
├── product_data_analysis_report.html     # generated report
└── order_data_analysis_report.html       # generated report
```
 
## Design overview
 
`DataProcessor<T>` is an abstract class implementing `DataProcessing<T>`, `DataReading<T>`, `DataStoring<T>` and `Analyzable<T>`. It provides the shared logic (CSV reading, writing, de-duplication, filtering, random sampling). Each concrete processor extends it and supplies the dataset-specific parts: line parsing, validity rules, CSV conversion, header and analysis.
 
`UserManager` implements `UserOperations` and uses small `Validator` implementations (`EmailValidator`, `PhoneNumberValidator`, `UserNameValidator`, `PasswordValidator`) for input checks.
 
## Requirements
 
- Java Development Kit (JDK) 8 or later (the code uses lambdas, streams and `Optional`)
- No external libraries or build tool are required
## Build and run
 
The source files declare the package `src.main.java`, so compile and run from the **project root** (the folder containing `src/` and `resources/`), because the CSV paths are relative.
 
**Linux / macOS**
 
```bash
javac -d out $(find src -name "*.java")
java -cp out src.main.java.Main
```
 
**Windows (Command Prompt)**
 
```bat
dir /s /b src\*.java > sources.txt
javac -d out @sources.txt
java -cp out src.main.java.Main
```
 
## Usage
 
1. At the start menu choose **1** to sign up or **2** to log in. Option **3** exits the program.
2. After a successful log-in, the data management menu offers:
   - **1** Manage customer data
   - **2** Manage product data
   - **3** Manage order data
   - **4** Manage all data
   - **5** Exit to the start menu
3. Each option runs the cleaning pipeline, prints the analysis, saves the processed sample to `resources/output*.csv`, and asks whether to generate the HTML report (`yes` / `no`).
Validation rules at sign-up:
 
| Field | Rule |
|---|---|
| Name | Not empty |
| Email | Standard `name@domain.tld` pattern |
| Phone number | 8–12 digits |
| Username | 3–20 characters: letters, digits, underscores; must be unique |
| Password | Letters and digits only, with at least one of each |
 
## Data format
 
| File | Columns |
|---|---|
| `customers.csv` | Customer_ID, Customer_Name, Email, Phone_Number, City, State, Country, Gender, Age_Group |
| `products.csv` | Product_ID, Product_Name, Category, Subcategory, Brand, Price |
| `order.csv` | Order_ID, Customer_ID, Product_ID, Seller_ID, Quantity, Order_Date (`yyyy-MM-dd`), Shipping_Cost, Discount_Amount, Payment_Method, Total_Amount, Delivery_Status, Review_Rating |
| `users.csv` | name, email, phone, username, password (no header row) |
 
Rows that do not have the expected number of columns are skipped. Files are split on commas, so values containing commas are not supported.
 
## Output
 
- `resources/outputcustomer.csv`, `outputproduct.csv`, `outputorder.csv` – the cleaned 200-record samples.
- `customer_data_analysis_report.html`, `product_data_analysis_report.html`, `order_data_analysis_report.html` – written to the directory the program is run from and overwritten on each run.
Because sampling is random, the figures differ from one run to the next.
 
## Scope and design decisions
 
DAMS is built as a lightweight, dependency-free console application, and its scope reflects that focus.
 
- **Dependency-free by design.** The project compiles with the plain JDK, so it runs anywhere Java is installed without a build tool or external libraries.
- **Sample-based analysis.** Each dataset is reduced to a random sample of 200 records, which keeps console output readable and runs fast on large CSV files. The sample size (`200`) is set in the three `processAndStore…Data` methods in `Main.java` and can be raised to analyse more data.
- **File-based storage.** Users and datasets live in plain CSV files, which keeps the data easy to inspect, edit and version without a database.
- **Straightforward CSV parsing.** Rows are split on commas, and rows with an unexpected column count are skipped, which matches the structure of the bundled datasets.
- **Report narratives.** The HTML reports present the computed figures alongside interpretive commentary written for the bundled dataset.
## Roadmap
 
Planned improvements to harden the project for production-style use:
 
1. **Credential security** – hash passwords (BCrypt or PBKDF2) instead of storing them in `resources/users.csv`. This is the top priority before the project handles any real user data.
2. **Dynamic report insights** – generate the commentary in the HTML reports from the computed results so it always matches the analysed sample.
3. **Robust parsing** – adopt a CSV library and add error handling for malformed numeric fields.
4. **Configurable sampling** – expose the sample size as a menu option or command-line argument, with an option to analyse the full dataset.
5. **Build and testing** – add a Maven or Gradle build and unit tests for the processors and validators.
