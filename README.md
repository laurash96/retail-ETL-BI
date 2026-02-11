# Retail ETL & BI Project

A comprehensive data engineering and business intelligence solution for retail customer analytics, featuring ETL pipelines, data enrichment, and actionable insights for decision-making.

**Author:** Laura Saldarriaga Higuita  
**Industry:** Retail (Textile)  
**Language:** Python  
**Tools:** Pandas, NumPy, Seaborn, Matplotlib, Looker Studio

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Business Context](#business-context)
- [Technical Architecture](#technical-architecture)
- [Data Sources](#data-sources)
- [ETL Pipeline](#etl-pipeline)
- [Data Cleaning & Quality](#data-cleaning--quality)
- [Feature Engineering](#feature-engineering)
- [Outputs & Deliverables](#outputs--deliverables)
- [Future Recommendations](#future-recommendations)
- [Setup & Installation](#setup--installation)

---

## 🎯 Project Overview

This project implements a complete ETL (Extract, Transform, Load) pipeline and feature engineering workflow for a retail textile company. The solution consolidates multiple data sources, performs extensive data cleaning, creates analytical features, and generates datasets ready for business intelligence visualization and decision-making.

### Key Achievements

- ✅ Integrated **4 data sources** into a unified dataset
- ✅ Processed **23.8M+ records** down to **4.8M unique transactions** after cleaning
- ✅ Created **7+ engineered features** for enhanced analytics
- ✅ Built **5 specialized datasets** for different analytical purposes
- ✅ Developed comprehensive BI dashboard in Looker Studio

---

## 💼 Business Context

### Problem Statement

The retail company needed to:
- Consolidate customer interaction data from multiple point-of-sale (PDV) locations
- Understand campaign effectiveness and customer behavior
- Enable data-driven decision-making for marketing and sales strategies

### Solution Delivered

A robust data pipeline that:
- Unifies customer contact data, transactions, product references, and campaigns
- Provides clean, enriched datasets for analysis
- Enables tracking of campaign performance and customer purchasing patterns

---

## 🏗️ Technical Architecture

### Technology Stack

```
Python 3.x
├── pandas         # Data manipulation and ETL
├── numpy          # Numerical operations
├── matplotlib     # Data visualization
└── seaborn        # Statistical visualizations
```

### Project Structure

```
retail-ETL-BI/
├── data_processing.ipynb      # Main ETL & Feature Engineering notebook
├── Report_Looker Studio.pdf   # BI Dashboard report
├── CASO PRACTICO EXCEL/       # Raw data sources (input)
│   ├── ClientesContactados_PDV_*.xlsx
│   ├── Movimientos.xlsx
│   ├── Referencias.xlsx
│   └── Campaigns.xlsx
└── NUEVOS DATOS/              # Processed data (output)
    ├── enriched_data_retail.csv
    ├── data_retail_nodates.csv
    ├── datos_campañas.csv
    ├── facturas_dias.csv
    └── clientes_ventas_contacto_campaña.csv
```

---

## 📊 Data Sources

### 1. Customer Contact Files (`ClientesContactados_PDV_*.xlsx`)
Multiple files containing customer contact information from different PDV locations.

**Key Fields:** Customer ID, Contact Type, Demographics (Age, Gender)

### 2. Transactions (`Movimientos.xlsx`)
Sales transaction records.

**Key Fields:** Invoice Number, Customer ID, Reference ID, Date, Sales, Units, Payment Method, Campaign Code

### 3. Product References (`Referencias.xlsx`)
Product catalog information.

**Key Fields:** Reference ID, Category, Group, Gender (target demographic)

### 4. Marketing Campaigns (`Campaigns.xlsx`)
Campaign metadata and timing.

**Key Fields:** Campaign Code, Start Date, End Date, Send Time

---

## 🔄 ETL Pipeline

### Phase 1: Data Extraction & Consolidation

#### Step 1: Merge Customer Contact Files
```python
# Technical Implementation
- Iterate through all PDV files (ClientesContactados_PDV_*.xlsx)
- Add PDV identifier column to each dataset
- Concatenate into single customer contact dataset
- Export: ClientesContactados.csv
```

**Records:** Multiple PDV files → Single consolidated customer table

#### Step 2: Join Transactions with Customer Data
```python
# Left join on Customer ID
merged = Movimientos + ClientesContactados
```

**Result:** Transaction records enriched with customer demographics

#### Step 3: Enrich with Product Information
```python
# Left join on Reference ID
merged = previous + Referencias
```

**Result:** Added product category, group, and gender targeting

#### Step 4: Add Campaign Data
```python
# Left join on Campaign Code
merged = previous + Campaigns
```

**Result:** Complete dataset with campaign timing information

**Output:** `merged_data_retail.csv` (23,843,522 records)

---

## 🧹 Data Cleaning & Quality

### 1. Negative Value Handling

**Issue Identified:**
- `UNIDADES`: 858,139 negative values
- `Ventas`: 1,065,878 negative values

**Solution:**
```python
# Remove records with negative sales or units
df = df[(df['Ventas'] >= 0) & (df['UNIDADES'] >= 0)]
```

**Impact:** Reduced to 22,777,644 valid records

---

### 2. Missing Value Treatment

#### Categorical Fields

| Field | Missing Values | Treatment | Rationale |
|-------|---------------|-----------|-----------|
| `PROMOCION` | Various | Fill with `'NO_PROMO'` | Indicate no active promotion |
| `TIPO CONTACTO` | Various | Fill with `'OTRO'` | Unknown contact type category |
| `CODIGO CAMPAÑA` | Various | Fill with `'DESCONOCIDO'` | Non-campaign transactions |

```python
df.loc[df['PROMOCION'].isna(), 'PROMOCION'] = 'NO_PROMO'
df.loc[df['TIPO CONTACTO'].isna(), 'TIPO CONTACTO'] = 'OTRO'
df.loc[df['CODIGO CAMPAÑA'].isna(), 'CODIGO CAMPAÑA'] = 'DESCONOCIDO'
```

---

### 3. Age Data Quality

**Challenge:** 61.24% of records had age = 0

**Analysis:**
- Used boxplot to identify distribution and outliers
- Calculated mean age from non-zero values: **38 years**
- Found 18,187 records with age > 100 (23 unique customers)

**Solution:**
```python
# Impute zeros with mean
mean_edad = 38
df['EDAD'] = df['EDAD'].replace(0, mean_edad)

# Cap unrealistic ages
df.loc[df['EDAD'] > 100, 'EDAD'] = mean_edad
```

---

### 4. Date Handling

**Missing Date Strategy:**
```python
# Set default date for missing values
default_date = pd.to_datetime('2000-01-01')
df['FECHA FACTURA'].fillna(default_date)
df['Fecha Inicio'].fillna(default_date)
df['Fecha Fin'].fillna(default_date)

# Default hour
df['Hora Envio'].fillna(0.0)
```

---

### 5. Duplicate Removal

```python
df = df.drop_duplicates()
df = df.reset_index(drop=True)
```

**Final Record Count:** 4,875,797 unique transactions

---

## 🔧 Feature Engineering

### 1. Campaign-to-Purchase Duration (`Dias campaña compra`)

**Purpose:** Measure response time from campaign start to purchase

```python
df['Dias campaña compra'] = (df['FECHA FACTURA'] - df['Fecha Inicio']).dt.days
```

**Business Value:** Identify optimal campaign duration and customer response patterns

---

### 2. Average Unit Price (`Precio promedio unidad`)

**Purpose:** Calculate price per unit for margin analysis

```python
df['Precio promedio unidad'] = df['Ventas'] / df['UNIDADES']
```

**Business Value:** Product pricing insights and revenue analysis

---

### 3. Purchase Day of Week (`Dia compra`)

**Purpose:** Identify weekly purchasing patterns

```python
df['Dia compra'] = df['FECHA FACTURA'].dt.day_name()

# Translate to Spanish
translation = {
    'Monday': 'Lunes', 'Tuesday': 'Martes', 
    'Wednesday': 'Miércoles', 'Thursday': 'Jueves',
    'Friday': 'Viernes', 'Saturday': 'Sábado', 
    'Sunday': 'Domingo'
}
df['Dia compra'] = df['Dia compra'].replace(translation)
```

**Business Value:** Optimize staffing and promotional timing

---

### 4. Campaign Window Flag (`En campaña`)

**Purpose:** Determine if purchase occurred during active campaign period

```python
df['En campaña'] = (
    (df['FECHA FACTURA'] >= df['Fecha Inicio']) & 
    (df['FECHA FACTURA'] <= df['Fecha Fin'])
).astype(int)
```

**Business Value:** Direct campaign attribution and ROI calculation

---

### 5. Days Since Last Purchase (`dias_ultima_compra`)

**Purpose:** Track customer purchase frequency and recency

```python
# Group by customer and calculate difference between purchases
grouped = df.groupby('ID CLIENTE')['FECHA FACTURA'].diff().dt.days
```

**Business Value:** 
- Customer retention analysis
- Churn prediction
- Reactivation campaign targeting

---

## 📦 Outputs & Deliverables

### Generated Datasets

| File | Records | Purpose |
|------|---------|---------|
| `enriched_data_retail.csv` | 4,875,797 | Complete enriched dataset with all features |
| `data_retail_nodates.csv` | Deduplicated | Sales data without campaign/contact timing |
| `datos_campañas.csv` | Per campaign | Campaign effectiveness metrics |
| `facturas_dias.csv` | Per invoice | Customer purchase frequency analysis |
| `clientes_ventas_contacto_campaña.csv` | Per customer | Customer-level aggregated metrics |
| `enriched_data_simp.csv` | 4,875,797 | Simplified dataset for movement counting |

---

### Key Metrics by Dataset

#### Campaign Analysis Dataset (`datos_campañas.csv`)
```
Fields:
- CODIGO CAMPAÑA
- Total Apariciones (total transactions)
- Si campaña (purchases during campaign)
- No campaña (purchases outside campaign)
- Porcentaje si campaña (campaign conversion %)
```

**Insight:** Quantifies which campaigns drove purchases during their active period

---

#### Customer Aggregation Dataset (`clientes_ventas_contacto_campaña.csv`)
```
Fields:
- ID CLIENTE
- Cantidad compras (purchase count)
- Ventas (total sales)
- EDAD (age)
- TIPO CONTACTO (contact methods used)
- CODIGO CAMPAÑA (campaigns received)
```

**Insight:** 360° customer view for segmentation and targeting

---

#### Purchase Frequency Dataset (`facturas_dias.csv`)
```
Fields:
- NUMERO FACTURA
- ID CLIENTE
- FECHA FACTURA
- CIUDAD PDV
- dias_ultima_compra (days since previous purchase)
```

**Insight:** Customer loyalty and retention metrics

---

## 📈 Analytical Insights

### Data Quality Improvements

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Records | 23,843,522 | 4,875,797 | Removed duplicates & invalid data |
| Negative values | 1.9M | 0 | 100% cleaned |
| Missing ages | 61.24% | 0% | Imputed with mean |
| Age outliers | 18,187 | 0 | Capped at realistic values |

---

### Campaign Performance Analysis

The pipeline enables answering:
- **What percentage of campaign transactions occurred during the campaign window?**
- **Which campaigns had the highest conversion rates?**
- **How many days after campaign start do customers typically purchase?**

---

### Customer Behavior Insights

The datasets support:
- **Purchase frequency:** Average days between purchases per customer
- **Channel effectiveness:** Which contact types drive the most sales
- **Weekly patterns:** Peak purchase days for inventory planning
- **Price sensitivity:** Unit price analysis by customer segment

---

## 🔮 Future Recommendations

### 1. Time Series Decomposition
Apply seasonal decomposition to identify:
- Long-term trends
- Seasonal patterns
- Cyclical behavior

**Suggested Method:** STL (Seasonal and Trend decomposition using Loess)

---

### 2. Correlation Analysis
Explore relationships between variables:
- Campaign type vs. purchase value
- Age/gender vs. product preferences
- Contact method vs. conversion rate

**Implementation:** Correlation matrix with heatmap visualization (partially implemented)

```python
correlation_matrix = numerical_columns.corr()
sns.heatmap(correlation_matrix, annot=True)
```

---

### 3. Customer Segmentation
Apply clustering algorithms to identify customer groups:

**Suggested Method:** K-Means clustering

**Segmentation Dimensions:**
- Purchase frequency
- Average transaction value
- Product category preferences
- Campaign responsiveness

---

### 4. Google Analytics Integration
For e-commerce transactions:
- Track user journey through purchase funnel
- Identify drop-off points
- A/B test campaign messaging

---

### 5. Predictive Modeling

#### a) Conversion Prediction (Logistic Regression)
If incomplete transaction data becomes available:
- Predict probability of purchase completion
- Identify at-risk transactions
- Enable proactive intervention

#### b) Demand Forecasting (ARIMA)
Time series forecasting for:
- Inventory planning
- Revenue projection
- Staffing optimization

---

## 🚀 Setup & Installation

### Prerequisites

```bash
Python 3.7+
Jupyter Notebook or JupyterLab
```

### Required Packages

```bash
pip install pandas numpy matplotlib seaborn openpyxl
```

### Running the Project

1. **Clone the repository:**
```bash
git clone <repository-url>
cd retail-ETL-BI
```

2. **Ensure data structure:**
```
retail-ETL-BI/
├── CASO PRACTICO EXCEL/       # Place source files here
└── NUEVOS DATOS/              # Output directory (auto-created)
```

3. **Execute the notebook:**
```bash
jupyter notebook data_processing.ipynb
```

4. **Run all cells sequentially** to:
   - Load and merge data sources
   - Clean and validate data
   - Engineer features
   - Generate output datasets

### Expected Runtime

- Full pipeline execution: ~15-30 minutes (depending on system specs)
- Processing 23M+ records requires ~4GB RAM

---

## 📊 BI Dashboard

The project includes a Looker Studio dashboard (`Report_Looker Studio.pdf`) featuring:

- Campaign performance KPIs
- Customer demographic breakdowns
- Sales trends and patterns
- Product category analysis
- Geographic performance (by CIUDAD PDV)

---

## 🤝 Contributing

This is portfolio project demonstrating ETL and data engineering capabilities for retail analytics.

---

## 📄 License

This project is part of a practical case study for educational and portfolio purposes.

---

## 👤 Contact

**Laura Saldarriaga Higuita**  
[GitHub Profile](https://github.com/laurasaldarriagahiguita)

---

## 🙏 Acknowledgments

- Data provided as part of retail analytics case study
- Visualization dashboard created with Google Looker Studio

---

## 📝 Technical Notes

### Data Privacy
- All customer IDs are anonymized/hashed
- No personally identifiable information (PII) is exposed

### Performance Considerations
- Large dataset processing may require optimization for production use
- Consider chunking for systems with limited memory
- Potential for parallelization using Dask or PySpark for scaling

### Code Quality
- Modular notebook structure with clear section headers
- Inline documentation in Spanish for business context
- Reproducible pipeline with consistent data transformations

---

*Last Updated: February 2026*

