# 🌿 Nutra Green Sales Analytics App
### Built with Qlik Cloud Analytics (QCA)

**Author:** Sabrina Puddu | Cloud Migration Specialist @ Qlik  

---

## 🎬 Demo Recording

> ** A full walkthrough demo recording is included in this repository.**
>
> The video covers a live narrated tour of all 4 dashboard sheets, the bookmark feature,
> and the 2024 New Customers story. 
> **Watch the demo before exploring the app** to understand the business context and key insights.
>
> 📹 `Nutra_Green_Sales_App_Demo.mp4`

---

## Overview

This repository contains the full deliverables for the **Nutra Green Sales Analytics App**.

**Nutra Green** is an eco-friendly chain of convenience stores undergoing a data analytics
migration. This app was designed to replace their legacy reporting system
with a fully interactive, self-service analytics solution - empowering every team to go
from raw data to actionable insights, all in one place.

The app spans **4 analytical sheets**, **1 bookmark**, and **1 story**, covering:
- Regional and country-level sales performance
- Product category analysis by customer type
- Year-on-Year growth tracking
- Customer profitability and segmentation
- A dedicated 2024 New Customers narrative story

---

## 📁 Repository Contents

| File | Description |
|---|---|
| `Nutra_Green_Sales_App.qvf` | Qlik application file - import directly into QCA |
| `Nutra_Green_Sales.pdf` | Exported stories |
| `Data_Model.png` |Screenshot of the Qlik data model |
| `Nutra_Green_Sales_App_Demo.mp4` | 📹 Full narrated demo recording (~10-12 min) |
| `README.md` | This file |

---

## Data Model

The app is built on a **star/snowflake hybrid schema** connecting 12 tables.
See `Data_Model.png` for the full visual.

### Tables & Key Fields

| Table | Key Fields | Description |
|---|---|---|
| `Orders` | Order Number 🔑, Customer Number, Employee Number | Transactional order header - Year, YearMonth, Invoice Date, Order Date |
| `OrderLineItems` | OrderLineKey 🔑, Item Number, Order Number | Line-level sales detail - Sales, Margin, LineSalesAmount, CostOfGoodsSold, GrossSales |
| `Customers` | Customer Number 🔑, Customer, Region | Customer master - address, country, region, geo coordinates |
| `Products` | Item Number 🔑, Manufacturer Number | Product hierarchy - Product Group, Sub Group, Item Desc, Unit Cost |
| `Manufacturers` | Manufacturer Number 🔑 | Supplier details - address, country, green rating |
| `ProductCostComparison` | Item Number 🔑 | Original unit cost for cost comparison analysis |
| `Employees` | Employee Number 🔑 | Sales rep data - job title, office location, specialty |
| `SalesPerson` | Employee Number 🔑 | Sales person name and title mapping |
| `Shipments` | OrderLineKey 🔑 | Shipment date linked per order line |
| `CustomerType` | Customer 🔑 | Classifies customers as **New** or **Existing** |
| `Population` | Region 🔑 | Region population - drives % Customers pie chart radius |
| `WeatherData` | Region 🔑 | Average temperature per region |

### Relationships

```
Orders ──────────────────── OrderLineItems       (via Order Number)
OrderLineItems ──────────── Products             (via Item Number)
OrderLineItems ──────────── Shipments            (via OrderLineKey)
Products ────────────────── Manufacturers        (via Manufacturer Number)
Products ────────────────── ProductCostComparison (via Item Number)
Orders ──────────────────── Customers            (via Customer Number)
Orders ──────────────────── Employees            (via Employee Number)
Employees ───────────────── SalesPerson          (via Employee Number)
Customers ───────────────── CustomerType         (via Customer)
Customers ───────────────── Population           (via Region)
Customers ───────────────── WeatherData          (via Region)
```

### Design Notes
- The **central fact table** is `OrderLineItems` - all sales metrics live here
- `Population` and `WeatherData` were loaded via **Data Manager** from `RegionPopulation.xlsx`
  and associated to the `Customers` table via the `Region` field
- `CustomerType` was loaded from `NutraGreenCustomerType.xlsx` and linked via `Customer`
- Geo fields (`City_GeoInfo`, `Country_GeoInfo`) power the **Sales by Country map** visualization
- `SalesPerson` is linked to `Employees` to enable sales rep-level analysis

---

## 📊 App Structure & Business Insights

### Sheet 1 - Regional Sales
> *"The starting point for any executive or regional manager who wants a quick pulse on the business."*

**Key visuals:**
- Sales & Margin by Region bar chart
- Sales KPI: **$32.69M** total | Margin KPI: **$18.92M** | Countries KPI: **21**
- Sales by Region pie chart
- % Customers pie chart (bubble radius driven by Population field)
- Sales by Country map - area layer colored by Sales master measure
- Regional Sales by Month and Year line chart (Aug 2022 → Feb 2025)

**Key Business Insights:**
- **EMEA dominates** with ~$15M in sales (45.3% of total) - nearly double Americas and South America
- Margin ratios are **consistent across all 3 regions**, suggesting a balanced global pricing strategy
- **EMEA represents 59.6% of the customer base** - concentration risk worth monitoring
- **USA and Brazil** are the top performing countries on the Sales by Country heatmap
- The time-series chart shows **EMEA consistently outperforms** other regions month over month
- A **notable upward trend in early 2025** signals accelerating growth momentum

---

### Sheet 2 - Product Sales Overview
> *"Where the product team and sales managers can really dig into the numbers."*

**Key visuals:**
- Year filter pane (2022-2025 self-service selection)
- Sales Total KPI: **683 customers**
- Sales, Margin and Order Volume by Product Group table (15 categories)
- Sales by New vs Existing Customers pie chart
- Sales by Product Group and Customer Type bar chart

**Key Business Insights:**
- **Alcoholic Beverages** leads all product categories at **$9.4M** in sales
- **Produce ($6.7M)** and **Canned Products ($3.8M)** follow as the next top categories
- **66.8% of sales come from New customers** vs only 33.2% from Existing - strong acquisition signal
- This ratio raises a **customer retention flag** - the business may need loyalty programs
- Alcoholic Beverages is **almost entirely driven by New customers**, warranting investigation
  into repeat purchase behavior for this category
- The Year filter allows instant slice-and-dice - select 2024 to isolate that year's performance

---

### Sheet 3 - Product Overview
> *"The strategic layer - combining treemap drill-down with Year-on-Year growth analysis."*

**Key visuals:**
- Sales by Region, Product Group, and Sub Group treemap (Top 5 products per region)
- **YoY Sales KPI: 142%** (2025 vs. 2024) - dynamic label
- Sales by Region and Product drill-down bar chart

**Master Dimension created:**
- `Products` drill-down: `Product Group` → `Product Sub Group` → `Item Desc`

**Master Measures created:**

*Sales PY (Previous Year Sales):*
```qlik
Sum({$<Year = {'$(=Max(Year)-1)'}, YearMonth=>} LineSalesAmount)
```

*YoY Sales (Year on Year Growth):*
```qlik
(Sum({$} LineSalesAmount) /
Sum({$<Year={'$(=Max(Year)-1)'}>} LineSalesAmount)) - 1
```

*Dynamic Label:*
```qlik
'YoY Sales ' & Max(Year) & ' vs. ' & (Max(Year)-1)
```

**Key Business Insights:**
- **YoY growth of 142% from 2024 to 2025** is an extraordinary signal of business momentum
- **EMEA drives Alcoholic Beverages** at 5M+ - the single largest product-region combination
- **Produce and Dairy dominate in EMEA** at the sub-group level
- In the Americas, **Produce and Alcoholic Beverages** lead
- South America is strongest in **Deli and Canned Products**
- The drill-down dimension allows users to click from Product Group all the way down
  to individual Item level - no filter panel needed

---

### Sheet 4 - Customer Analysis
> *"The most actionable sheet - built for account managers and customer success teams."*

**Key visuals:**
- Number of Customers KPI: **683**
- Customer and Country filter panes
- Top 10 Customers Based on Sales table (with Others row)
- Sales and Avg COGS by Customer scatter plot - Top 10 (bubble size = Order Volume, color = Region)
- Sales by Customer and Product Group bar chart - Top 5 (red reference line at $80k)

**Key measures:**
| Measure | Expression | Used In |
|---|---|---|
| Avg COGS | `Avg(CostOfGoodsSold)` | Scatter plot X axis |
| Num Customers | `Count(Distinct [Customer Number])` | KPI |
| Order Volume | `Count(Distinct [Order Number])` | Scatter bubble size, table |

**Key Business Insights:**
- **Target leads** at **$1.5M** in sales but with only **42 orders** - a large, infrequent buyer
  presenting supply chain and account dependency risk
- **Userland at $1.39M** with only **12 orders** - an extreme concentration risk requiring
  dedicated account management
- **All Top 5 customers far exceed the $80k Sales Target** - the reference line confirms
  strong performance at the top of the customer pyramid
- The **scatter plot reveals 4-dimensional profitability** - sales, COGS, order volume,
  and region in a single view. Customers in the upper-right quadrant (high sales, higher COGS)
  warrant margin management attention
- **Karsing** has 324 order volumes but lower total sales - a high-frequency, lower-value
  account profile worth understanding for operational efficiency

---

## 🔖 Bookmark

**Regional Sales by Month and Year**

| Property | Value |
|---|---|
| Selection saved | January 2024 - March 2024 |
| Primary use case | EMEA regional Q1 performance review |
| Behavior | All charts, KPIs and map update simultaneously on apply |

> **Pro tip for business users:** If you only need to focus on a specific region -
> for example EMEA - use this bookmark instead of manually reapplying filters each session.
> One click applies the entire saved selection across every visual on the sheet.
> This is Qlik's **associative engine** working in real time.

---

##Story - Nutra Green Story #1

### 2024 New Customers Analysis
*Selections applied: Year = 2024 | Customer Type = New*

Built using QCA's **snapshot and storytelling** feature - all slides were captured with
active selections, making this presentation-ready without any external tools.

---

#### Slide 1 - 2024 New Customers Analysis

**Visuals:** Sales by Country map | Sales & Margin by Region bar chart | Customers KPI | Margin KPI

**Insights:**
- **332 new customers** were acquired in 2024 - a strong cohort
- Generated **$5.33M in margin** from new customers alone
- **EMEA leads new customer acquisition** with ~$5M in sales
- The USA and Germany are the darkest on the heatmap - top new customer markets
- Brazil shows significant new customer activity in South America
- Margin tracks proportionally across all regions - **no region is being discounted to win new business**

---

#### Slide 2 - 2024 Product Analysis

**Visuals:** Sales by Region, Product Group and Sub Group treemap | Sales by Product Group bar chart (New only)

**Insights:**
- In EMEA, **Produce dominates** new customer purchases - Fresh Vegetables at 320k, Hot Dogs at 225k
- In Americas, **Dairy and Deli** are the preferred categories for new customers
- South America new customers gravitate toward **Deli and Canned Products**
- **Alcoholic Beverages is the #1 product group** for new customers at ~$3M - consistent with
  the overall app trend and reinforcing it as a key customer acquisition driver
- **Produce and Frozen Foods** follow as second and third - important for inventory planning
- These regional product preferences are **actionable for merchandising teams**:
  stock the right products in the right regions to maximize new customer conversion

---

#### Slide 3 - 2024 Customer Overview

**Visuals:** YoY Sales KPI | Top 10 New Customers table | Sales by Customer and Product Group bar chart

**Insights:**
- **YoY Sales 2024 vs. 2023: +7%** - solid, sustainable growth in the new customer segment
- This 7% in 2024 laid the foundation for the **explosive 142% growth seen in 2025**
- **Matradi leads** new customers at **$840,000** - a significant account
- Acer ($508k) and Boston and Albany Railroad Company ($418k) follow
- Total new customer sales: **$9.27M** across 9,034 order volumes
- **Target spikes** in the bar chart driven by a single product group -
  flagging an account diversification conversation with the sales team
- All Top 5 new customers exceed the **$80k Sales Target** reference line

---

## How to Use This App

### Prerequisites
- Active Qlik Cloud Analytics (QCA) account
- Personal or shared Space available

### Import Steps
1. Log in to your Qlik Cloud tenant
2. Click **+ Create** → **Upload app**
3. Select `Nutra_Green_Sales.qvf`
4. Set Space to **Personal** → click **Upload**
5. Upload all supporting Excel data files to your Personal space via **+ Create → Dataset**
6. Open the app and click **Reload data**
7. Navigate through the 4 sheets using the arrow controls at the top right

### Recommended Viewing Order
1. 🌍 **Regional Sales** - start here for the big picture
2. 📦 **Product Sales Overview** - use the Year filter to explore by year
3. 🗺️ **Product Overview** - click into the treemap to drill down
4. 👥 **Customer Analysis** - use the Customer/Country filters to focus on segments
5. 📖 **Story** - open from the hamburger menu → Stories → Nutra Green Story #1

---


## QCA Features Showcased

| Feature | Where Used |
|---|---|
| **Master Dimensions** | Customer Type, Products drill-down, Year |
| **Master Measures** | Sales, Margin, Order Volume, Sales CY, Sales PY, YoY Sales, Avg COGS, Num Customers |
| **Set Analysis** | Sales PY and YoY Sales expressions |
| **Dynamic Labels** | YoY Sales KPI - shows current and prior year dynamically |
| **Map Visualization** | Sales by Country - area layer colored by Sales master measure |
| **Drill-down Dimension** | Product Group → Sub Group → Item Desc |
| **Data Manager** | RegionPopulation.xlsx loaded and associated via Region field |
| **Bookmarks** | Regional Sales by Month and Year - Q1 2024 EMEA selection |
| **Storytelling & Snapshots** | 2024 New Customers Analysis - 3 slides |
| **Filter Panes** | Year, Customer, Country |
| **Reference Lines** | $80k Sales Target in Customer bar chart |
| **Pie Chart Radius** | % Customers radius driven by Max(Population) |
| **Scatter Plot** | 4-dimensional customer view - Sales, Avg COGS, Order Volume, Region |

---

## Author

**Sabrina Puddu**
Cloud Migration Specialist @ Qlik

---

