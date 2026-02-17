# 🚔 Vehicle Crime Analytics: SQL Portfolio Project

[![SQL](https://img.shields.io/badge/SQL-SQLite-blue)](https://www.sqlite.org/)
[![Python](https://img.shields.io/badge/Python-3.10+-green)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange)](https://jupyter.org/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **End-to-end SQL data investigation simulating a junior data analyst role for the Nigerian Police Force**

A comprehensive SQL portfolio project demonstrating data analysis capabilities through real-world crime reporting scenarios. This project showcases advanced SQL techniques, analytical reasoning, and the ability to translate raw data into actionable business insights.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Key Insights](#-key-insights)
- [Technical Skills Demonstrated](#-technical-skills-demonstrated)
- [Database Schema](#-database-schema)
- [Analytical Questions](#-analytical-questions)
- [Installation & Setup](#-installation--setup)
- [Project Structure](#-project-structure)
- [Sample Queries](#-sample-queries)
- [Results & Recommendations](#-results--recommendations)
- [Ethical Considerations](#-ethical-considerations)
- [Future Work](#-future-work)
- [Author](#-author)

---

## 🎯 Project Overview

### Context
Acting as a **Junior Data Analyst** for the Nigerian Police Force's Central Motor Registry (CMR), this project analyzes historical vehicle crime reports to uncover patterns, support investigations, and inform policy decisions.

### Objectives
- ✅ Identify high-risk vehicle owners and geographic hotspots
- ✅ Analyze investigator workload distribution for resource optimization
- ✅ Detect repeat victims requiring targeted intervention
- ✅ Distinguish organized crime patterns from opportunistic theft
- ✅ Provide data-driven recommendations for prevention strategies

### Dataset
- **150 crime reports** spanning 2023-2025
- **200 vehicles** across multiple makes/models
- **100 individuals** (owners and reporters)
- **50 investigators** across 20 geographic zones
- **5 interconnected tables** with relational constraints

---

## 💡 Key Insights

### 1. 🎯 Repeat Victim Crisis
**39 individuals** experienced multiple vehicle thefts (up to 8 incidents per person), representing **50+ preventable future incidents** through targeted intervention.

### 2. 📍 Geographic Patterns Suggest Organized Crime
Analysis of incidents-per-owner ratios reveals three distinct crime types:
- **Repeat Targeting Zones** (e.g., Enugu Zone 12: 1.40 incidents/owner)
- **Transit Hubs** (high distances, diverse vehicles)
- **Opportunistic Crime Zones** (standard patterns)

### 3. ⚖️ Investigator Workload Imbalance
Caseload ranges from **1 to 7 cases per investigator**, suggesting need for workload rebalancing or recognition of differential jurisdictional crime rates.

### 4. 🔄 High Abandonment Rate
**51 of 150 cases (34%)** result in "Abandoned" status, indicating potential gaps in follow-through or investigation resources.

---

## 🛠️ Technical Skills Demonstrated

### SQL Proficiency
- ✅ **Complex Joins**: 4-table INNER JOINs for comprehensive reporting
- ✅ **Aggregation**: GROUP BY, COUNT, AVG with HAVING clauses
- ✅ **Window Functions**: ROW_NUMBER() with PARTITION BY for ranking
- ✅ **Common Table Expressions (CTEs)**: Multi-step query logic
- ✅ **Subqueries**: Nested SELECT statements for filtered datasets
- ✅ **Self-Joins**: Finding latest records per entity

### Analytical Capabilities
- 📊 Pattern recognition and anomaly detection
- 📈 Workload and resource distribution analysis
- 🗺️ Geographic and spatial trend identification
- 🔁 Repeat event and time-series analysis
- 💼 Business impact framing and recommendation development

### Soft Skills
- 📝 Clear technical documentation for non-technical stakeholders
- 🤔 Critical thinking about data limitations and bias
- ⚖️ Ethical awareness in crime data interpretation
- 🎯 Strategic prioritization of insights by business value

---

## 🗂️ Database Schema

### Entity Relationship Diagram

```
individuals (owners/reporters)
    │
    ├─< cars (1:M)
    │     │
    │     └─< reports (1:M)
    │           │
    │           ├─> investigators (M:1)
    │           └─> locations (M:1) [last_seen]
    │
    └─> locations (M:1) [owner_location]

investigators
    └─> locations (M:1) [jurisdiction]
```

### Table Descriptions

**`individuals`** - Vehicle owners and crime reporters
- Primary Key: `id`
- Foreign Key: `location_id` → `locations.id`
- Contains: Personal details, contact info, home location

**`cars`** - Vehicle registration data
- Primary Key: `id`
- Foreign Key: `individual_id` → `individuals.id`
- Contains: Make, model, year, color, license plate

**`reports`** - Crime incident records
- Primary Key: `id`
- Foreign Keys: `car_id`, `reporter_individual_id`, `investigator_id`, `last_seen_location_id`
- Contains: Status, timestamps, distance from owner, notes

**`investigators`** - Police personnel
- Primary Key: `id`
- Foreign Key: `jurisdiction_location_id` → `locations.id`
- Contains: Name, badge number, jurisdiction, contact

**`locations`** - Geographic zones
- Primary Key: `id`
- Contains: Zone name, city, state, coordinates

---

## ❓ Analytical Questions

| # | Question | SQL Techniques |
|---|----------|----------------|
| 1 | Top 10 vehicle owners by car count | GROUP BY, ORDER BY, LIMIT |
| 2 | Owners with > 3 vehicles | HAVING clause filtering |
| 3 | Recent reports with full details | 4-table INNER JOIN |
| 4 | Report distribution by status | Simple aggregation |
| 5 | Investigator workload analysis | Multi-table JOIN + GROUP BY |
| 6 | Latest report per vehicle | Subquery with MAX() |
| 7 | Average distance by report status | AVG() aggregation, ROUND() |
| 8 | Repeat victim identification | Chained JOINs, HAVING > 1 |
| 9 | Top 3 vehicles per location | CTE + ROW_NUMBER() window function |
| 10 | First case per investigator | CTE + PARTITION BY + ranking |

### Capstone Analysis
**Original Research Question**: *"Does geographic clustering suggest organized theft rings versus opportunistic crime?"*

Developed custom classification system based on:
- Incidents-per-owner ratio (repeat targeting indicator)
- Abandoned vehicle rate (chop shop indicator)
- Average distance from owner (transit hub indicator)

---

## 🚀 Installation & Setup

### Prerequisites
```bash
Python 3.8+
Jupyter Notebook
SQLite3
```

### Quick Start
```bash
# Clone repository
git clone https://github.com/yourusername/vehicle-crime-analytics.git
cd vehicle-crime-analytics

# Install dependencies
pip install jupyter pandas ipython-sql

# Launch Jupyter Notebook
jupyter notebook sql_crime_analytics_COMPLETED.ipynb
```

### Database Setup
The SQLite database (`crime_reports__3_.sqlite`) is included in the repository. No additional setup required.

---

## 📁 Project Structure

```
vehicle-crime-analytics/
│
├── sql_crime_analytics_COMPLETED.ipynb    # Complete solution notebook
├── crime_reports__3_.sqlite               # SQLite database
├── SQL_Crime_Analytics_Solutions.md       # Detailed explanations
├── README.md                              # This file
│
└── assets/
    └── erd_diagram.png                    # Database schema visualization
```

---

## 📊 Sample Queries

### Query 1: Top Vehicle Owners
```sql
SELECT 
    i.id AS individual_id,
    i.first_name || ' ' || i.last_name AS owner,
    COUNT(c.id) AS cars_owned
FROM individuals i
INNER JOIN cars c ON i.id = c.individual_id
GROUP BY i.id, i.first_name, i.last_name
ORDER BY cars_owned DESC
LIMIT 10;
```

**Result**: Identified Karen Torres with 8 vehicles as highest-risk fleet owner.

---

### Query 8: Repeat Victims
```sql
SELECT 
    i.id AS owner_id,
    i.first_name,
    i.last_name,
    COUNT(r.id) AS total_reports
FROM individuals i
INNER JOIN cars c ON i.id = c.individual_id
INNER JOIN reports r ON c.id = r.car_id
GROUP BY i.id, i.first_name, i.last_name
HAVING COUNT(r.id) > 1
ORDER BY total_reports DESC;
```

**Result**: Kevin Flores experienced 8 separate theft incidents, highlighting critical need for victim support programs.

---

### Query 9: Geographic Hotspots (Advanced)
```sql
WITH ranked_reports AS (
    SELECT 
        l.name AS location,
        c.plate,
        COUNT(r.id) AS reports_count,
        ROW_NUMBER() OVER (
            PARTITION BY l.name 
            ORDER BY COUNT(r.id) DESC, c.plate ASC
        ) AS rn
    FROM reports r
    INNER JOIN cars c ON r.car_id = c.id
    INNER JOIN locations l ON r.last_seen_location_id = l.id
    GROUP BY l.name, c.plate
)
SELECT location, plate, reports_count
FROM ranked_reports
WHERE rn <= 3
ORDER BY location, reports_count DESC;
```

**Result**: Identified Abuja Zone 11 plate RHB-0442 with 2 reports as highest single-vehicle concentration.

---

## 📈 Results & Recommendations

### Priority 1: Repeat Victim Program
**Target**: 39 identified repeat victims  
**Action**: Proactive outreach with free vehicle tracking, security consultation, insurance liaison  
**Expected Impact**: 50+ prevented future incidents

### Priority 2: Three-Tiered Geographic Strategy

**Tier 1: Organized Crime Zones** (incidents/owner > 1.5)
- Deploy undercover operations
- Install automated license plate readers
- Cross-jurisdictional coordination

**Tier 2: Transit/Chop Shop Zones** (high abandonment + high distance)
- Increase hub/border patrols
- Partner with scrap dealers for stolen vehicle ID
- Mandatory documentation checks at repair shops

**Tier 3: Opportunistic Crime Zones** (standard patterns)
- Community awareness campaigns
- Environmental design improvements (lighting, CCTV)
- Subsidized vehicle tracking for high-value owners

### Priority 3: Workload Rebalancing
**Analysis**: 7:1 caseload ratio (highest:lowest investigators)  
**Action**: Quarterly workload audits, case redistribution protocol  
**Expected Impact**: Improved investigator morale, faster case resolution

---

## ⚖️ Ethical Considerations

### Data Limitations Acknowledged
- **Selection Bias**: Only reported crimes captured (excludes unreported incidents)
- **Geographic Bias**: Higher reporting may reflect infrastructure, not actual crime rates
- **Socioeconomic Factors**: High-income owners overrepresented due to insurance requirements

### Responsible Data Use
- ✅ Aggregate data to protect individual privacy
- ✅ Avoid stigmatizing communities as "high-crime zones"
- ✅ Frame findings as prevention opportunities, not enforcement justification
- ✅ Acknowledge potential for algorithmic bias in geographic targeting
- ✅ Recommend parallel social service investment in affected areas

### Privacy Protection
No personally identifiable information (PII) is exposed in public-facing analysis. Repeat victim outreach should be conducted confidentially to prevent further targeting.

---

## 🔮 Future Work

### 1. Temporal Pattern Analysis
Add time-series analysis for seasonal/hourly crime patterns to enable predictive patrol scheduling.

### 2. Vehicle Risk Profiling
Calculate theft rates per 1000 vehicles by make/model, controlling for market share.

### 3. Investigation Effectiveness Study
Logistic regression to identify factors predicting successful vehicle recovery (Found status).

### 4. Geospatial Enhancement
Integrate latitude/longitude for road distance calculations and heat map visualizations.

### 5. Dashboard Development
Build Power BI dashboard for real-time monitoring by stakeholders.

---

## 👨‍💻 Author

**[Your Name]**  
*Economic Research & Data Analyst*

- 🔗 LinkedIn: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)
- 💼 Portfolio: [yourwebsite.com](https://yourwebsite.com)
- 📧 Email: your.email@example.com
- 🐙 GitHub: [@yourusername](https://github.com/yourusername)

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Project framework provided by [Course/Bootcamp Name]
- Database design inspired by real-world law enforcement systems
- Special thanks to instructors and peers for feedback and collaboration

---

## 📢 Feedback & Contact

Found a bug? Have a suggestion? Want to collaborate?

- **Open an Issue**: [GitHub Issues](https://github.com/yourusername/vehicle-crime-analytics/issues)
- **Start a Discussion**: [GitHub Discussions](https://github.com/yourusername/vehicle-crime-analytics/discussions)
- **Connect**: Let's discuss data analytics on [LinkedIn](https://linkedin.com/in/yourprofile)

---

<div align="center">

**⭐ If you found this project helpful, please consider giving it a star! ⭐**

[![GitHub Stars](https://img.shields.io/github/stars/yourusername/vehicle-crime-analytics?style=social)](https://github.com/yourusername/vehicle-crime-analytics)
[![GitHub Forks](https://img.shields.io/github/forks/yourusername/vehicle-crime-analytics?style=social)](https://github.com/yourusername/vehicle-crime-analytics/fork)

</div>
