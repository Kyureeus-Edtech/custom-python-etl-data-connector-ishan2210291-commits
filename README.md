# 🧩 Software Architecture - Assignment 2

## Custom Python ETL Data Connector

**Student:** Ishan Pandita
**Register No:** 3122225001042
**Department:** CSE  
**Section:** A
**Connector:** CZDS (ICANN) - Centralized Zone Data Service  
**Institution:** SSN College of Engineering

---

## 📖 **Assignment Objective**

The objective of this assignment is to design and implement a **Custom Python ETL Data Connector** that interacts with an external data source (API), processes the extracted data, and loads it into a **MongoDB database**.  
This connector demonstrates secure authentication handling, multi-endpoint API usage, and data transformation techniques, following good software architecture and ETL design principles.

---

## ⚙️ **Chosen Data Source: ICANN - Centralized Zone Data Service (CZDS)**

The **Centralized Zone Data Service (CZDS)** API, maintained by **ICANN (Internet Corporation for Assigned Names and Numbers)**, provides programmatic access to DNS zone data of approved top-level domains (TLDs).

The connector uses **three different endpoints** from the same data source as per the assignment requirements.

---

## 🌐 **API Endpoints Used**

| Step | Endpoint                     | Description                                                                                                     | HTTP Method |
| ---- | ---------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------- |
| 1️⃣   | `/api/authenticate`          | Authenticates user and retrieves an **access token** for further API calls                                      | `POST`      |
| 2️⃣   | `/czds/downloads/links`      | Lists all **authorized zone file download URLs** for the authenticated user                                     | `GET`       |
| 3️⃣   | `/czds/downloads/{tld}.zone` | Retrieves **metadata of zone files** (e.g., file size, last modified date) without downloading the full content | `HEAD`      |

These endpoints collectively implement the **Extract** stage of the ETL pipeline.

---

## 🔄 **ETL Workflow**

### **1. Extract Phase**

- The connector first **authenticates** with the ICANN CZDS API using `/api/authenticate`.
- After successful login, it retrieves a list of authorized **zone file links** using `/czds/downloads/links`.
- Finally, it collects metadata (size, timestamp) for a few selected zones using `/czds/downloads/{tld}.zone`.

### **2. Transform Phase**

- Each zone’s metadata is processed to:
  - Convert **file size from bytes → kilobytes (KB)**.
  - Add a **category label** (`LARGE` or `SMALL`) based on the file size.
  - Include a timestamp of when the record was fetched.

### **3. Load Phase**

- The transformed data is inserted into a **MongoDB Atlas collection** named `czds_icann_connector`.
- Each run creates a new document containing:
  - Access status
  - Timestamp of retrieval
  - Zone metadata list
  - Total number of zones processed

---

## 🧠 **Data Transformation Example**

| Zone | File Size (bytes) | File Size (KB) | Category | Last Modified                 |
| ---- | ----------------- | -------------- | -------- | ----------------------------- |
| com  | 3720              | 3.63           | SMALL    | Mon, 04 Mar 2019 19:45:43 GMT |
| org  | 4100              | 4.00           | LARGE    | Tue, 05 Mar 2019 18:30:12 GMT |
| net  | 3900              | 3.81           | LARGE    | Wed, 06 Mar 2019 14:20:55 GMT |

---

## 📞 **Database Details**

**MongoDB Cluster URI:**

```
mongodb+srv://interntracker:interntracker123@interntracker-cluster.j6czuyl.mongodb.net/?retryWrites=true&w=majority
```

**Database Name:** `ssn_arch_assign2`  
**Collection Name:** `czds_icann_connector`

Each ETL run inserts a new document similar to:

```json
{
  "retrieved_at": "2025-10-20T10:30:00Z",
  "token_status": "success",
  "total_zones": 3,
  "zones": [
    {
      "zone": "com",
      "size_kb": 3.63,
      "last_modified": "Mon, 04 Mar 2019 19:45:43 GMT",
      "category": "SMALL"
    },
    {
      "zone": "org",
      "size_kb": 4.0,
      "last_modified": "Tue, 05 Mar 2019 18:30:12 GMT",
      "category": "LARGE"
    },
    {
      "zone": "net",
      "size_kb": 3.81,
      "last_modified": "Wed, 06 Mar 2019 14:20:55 GMT",
      "category": "LARGE"
    }
  ]
}
```

---

## 🧮 **Project Files and Structure**

```
/Ishu_czds_connector/
│
├── etl.py                 # Main ETL pipeline script
├── .env                   # Stores MongoDB credentials (not committed)
├── requirements.txt        # Dependencies
└── README.md               # Documentation
```

---

## 🔐 **.env File Configuration**

```bash
MONGO_URI=mongodb+srv://interntracker:interntracker123@interntracker-cluster.j6czuyl.mongodb.net/?retryWrites=true&w=majority
MONGO_DB=ssn_arch_assign2
```

---

## 🧩 **Dependencies**

Install the following Python libraries before running the connector:

```
pymongo
python-dotenv
```

You can install them using:

```bash
pip install -r requirements.txt
```

---

## ▶️ **How to Run**

1. Clone your branch from the official repository:

   ```bash
   git clone https://github.com/Kyureeus-Edtech/kyureeus-ssn-software-architecture-assignment-custom-python-etl-data-connector-SSN-college-software-
   ```

2. Navigate to your connector folder:

   ```bash
   cd ishu_czds_connector
   ```

3. Create a `.env` file with your MongoDB details (as shown above).

4. Run the connector:

   ```bash
   python etl.py
   ```

5. Verify data insertion in MongoDB Atlas under the `czds_icann_connector` collection.

---

## 📊 **Console Output Example**

```
✅ Connected to MongoDB Database: ssn_arch_assign2
================================================================================

🔹 [ENDPOINT 1] /api/authenticate → Requesting authentication token...
✅ Access token received successfully.
👤 User:ishu@ssn.edu.in

🔹 [ENDPOINT 2] /czds/downloads/links → Retrieving authorized zone file URLs...
✅ Retrieved zone file URLs:
   - https://czds-download-api.icann.org/czds/downloads/com.zone
   - https://czds-download-api.icann.org/czds/downloads/org.zone
   - https://czds-download-api.icann.org/czds/downloads/net.zone

🔹 [ENDPOINT 3] /czds/downloads/{tld}.zone → Fetching zone file metadata...
✅ COM → Size: 3720 bytes | Modified: Mon, 04 Mar 2019 19:45:43 GMT
✅ ORG → Size: 4100 bytes | Modified: Tue, 05 Mar 2019 18:30:12 GMT
✅ NET → Size: 3900 bytes | Modified: Wed, 06 Mar 2019 14:20:55 GMT

⚙️  Transforming extracted data for MongoDB storage...
✅ Transformation completed. Summary:
   COM | 3.63 KB | SMALL
   ORG | 4.0 KB | LARGE
   NET | 3.81 KB | LARGE

💾 Data successfully inserted into MongoDB collection: czds_icann_connector
✅ ETL Pipeline executed successfully.
================================================================================
```

---

## 🧮 **Highlights of the Implementation**

- ✅ Uses **three unique API endpoints** from the same data source (ICANN CZDS).
- ✅ Demonstrates **Extract → Transform → Load** stages clearly.
- ✅ Implements **data cleaning and categorization** before loading.
- ✅ Uses **environment variables** for secure connection handling.
- ✅ Fully compatible with **MongoDB Atlas** and Python 3.x.
- ✅ Clean and modular code structure following **software architecture best practices**.

---

## 🏁 **Conclusion**

This ETL connector demonstrates how a software system can securely interact with a REST API, extract structured data from multiple endpoints, transform it for consistency and analysis, and efficiently load it into a NoSQL database.  
It serves as a practical example of applying **software architecture principles, API integration, and data engineering concepts** in a real-world environment.
