# Vehicle Crime Analytics: SQL Portfolio Project Solutions

**Analyst:** Economic Research and Data Analyst  
**Database:** Nigerian Police Force Central Motor Registry (CMR)  
**Project Type:** Portfolio SQL Analysis

---

## Database Schema Overview

### Tables and Relationships

**cars** - Vehicle registration information
- id, individual_id (FK), make, model, year, color, plate, created_at

**individuals** - Vehicle owners and reporters
- id, first_name, last_name, gender, age, phone, email, location_id (FK), created_at

**investigators** - Police investigators
- id, first_name, last_name, badge_number, jurisdiction_location_id (FK), phone, created_at

**locations** - Geographic zones
- id, name, city, state, latitude, longitude

**reports** - Vehicle theft/crime reports
- id, car_id (FK), reporter_individual_id (FK), investigator_id (FK), status, last_seen_location_id (FK), distance_from_owner_km, reported_at, created_at, notes

---

## Question 1: Count number of cars per individual and show top 10 owners

### SQL Query
```sql
%%sql q1_result <<
SELECT 
    i.id AS individual_id,
    i.first_name || ' ' || i.last_name AS owner,
    COUNT(c.id) AS cars_owned
FROM individuals i
INNER JOIN cars c ON i.id = c.individual_id
GROUP BY i.id, i.first_name, i.last_name
ORDER BY cars_owned DESC, individual_id ASC
LIMIT 10;
```

### Explanation
This query identifies the top vehicle owners in the database, which helps prioritize outreach for fleet security awareness programs. By joining the individuals and cars tables on the owner relationship, we aggregate the count of vehicles per person. The INNER JOIN ensures we only count individuals who actually own vehicles. Results are ordered by cars_owned in descending order to surface the highest-risk fleet owners first, with a secondary sort by individual_id for consistency. The LIMIT 10 focuses attention on the most significant cases. This reveals that the top owner has 8 vehicles, suggesting a need for targeted fleet management guidance for individuals with 5+ vehicles.

---

## Question 2: List individuals who own more than 3 cars

### SQL Query
```sql
%%sql q2_result <<
SELECT 
    i.id AS individual_id,
    i.first_name,
    i.last_name,
    COUNT(c.id) AS cars_owned
FROM individuals i
INNER JOIN cars c ON i.id = c.individual_id
GROUP BY i.id, i.first_name, i.last_name
HAVING COUNT(c.id) > 3
ORDER BY cars_owned DESC, individual_id ASC;
```

### Explanation
This query filters for individuals with significant vehicle portfolios (more than 3 cars), which represents a higher-value target segment for theft prevention programs. The query structure is similar to Question 1, but uses the HAVING clause to apply a post-aggregation filter specifically for owners with more than 3 vehicles. This filtering approach is necessary because WHERE clauses cannot evaluate aggregate functions. The results show 21 individuals meeting this criterion, representing concentrated vehicle ownership that warrants specialized security consultation. This segment likely includes small business owners, fleet operators, or collectors who would benefit from enhanced tracking and insurance protocols.

---

## Question 3: Get recent reports with car, owner and report status

### SQL Query
```sql
%%sql q3_result <<
SELECT 
    r.id AS report_id,
    r.reported_at,
    r.status,
    c.plate,
    c.make,
    c.model,
    i.first_name,
    i.last_name,
    l.name AS last_seen_location
FROM reports r
INNER JOIN cars c ON r.car_id = c.id
INNER JOIN individuals i ON c.individual_id = i.id
INNER JOIN locations l ON r.last_seen_location_id = l.id
ORDER BY r.reported_at DESC
LIMIT 20;
```

### Explanation
This query provides a comprehensive snapshot of the most recent vehicle crime cases, joining four tables to create an actionable report for daily operations briefings. The multiple INNER JOINs connect reports to vehicles (via car_id), vehicles to owners (via individual_id), and reports to locations (via last_seen_location_id). Ordering by reported_at DESC ensures the most recent cases appear first, which is critical for prioritizing active investigations. The 20-row limit keeps the report digestible for quick morning briefings. The output reveals status distribution and geographic patterns, showing that cases span multiple zones with various outcomes including Abandoned, Found, Pending Investigation, and Not Assigned statuses.

---

## Question 4: Count reports per status

### SQL Query
```sql
%%sql q4_result <<
SELECT 
    status,
    COUNT(*) AS reports_count
FROM reports
GROUP BY status
ORDER BY reports_count DESC;
```

### Explanation
This query quantifies the distribution of case statuses, providing a high-level operational metric for resource allocation and process improvement. The simple GROUP BY aggregation on the status field reveals workload distribution across case types. The ORDER BY reports_count DESC highlights the most common status first, helping leadership understand where bottlenecks or resource gaps exist. Results show that "Abandoned" cases (51) represent the largest category, followed by "Found" (42) and "Pending Investigation" (41), with "Not Assigned" cases being relatively rare (16). This suggests potential issues with case follow-through or a need for additional investigative resources for the high volume of abandoned vehicle cases.

---

## Question 5: Investigator workload (number of reports assigned)

### SQL Query
```sql
%%sql q5_result <<
SELECT 
    inv.id,
    inv.first_name,
    inv.last_name,
    COUNT(r.id) AS assigned_reports,
    l.name AS jurisdiction
FROM investigators inv
INNER JOIN reports r ON inv.id = r.investigator_id
INNER JOIN locations l ON inv.jurisdiction_location_id = l.id
GROUP BY inv.id, inv.first_name, inv.last_name, l.name
ORDER BY assigned_reports DESC, inv.id ASC;
```

### Explanation
This query analyzes workload distribution across investigators, revealing potential resource imbalances and informing case assignment decisions. By joining investigators to their assigned reports and their jurisdictional locations, we create a complete picture of each investigator's caseload within their geographic area of responsibility. The GROUP BY ensures we count reports per investigator, while the ORDER BY reveals who has the highest workload. The data shows Kenneth White (Port Harcourt Zone 4) leads with 7 cases, while several investigators have only 1-2 cases, suggesting an uneven distribution. This insight could drive workload rebalancing initiatives or indicate that some zones experience higher crime rates than others.

---

## Question 6: Latest report per car

### SQL Query
```sql
%%sql q6_result <<
SELECT 
    r.id AS report_id,
    r.car_id,
    r.reported_at,
    r.status
FROM reports r
INNER JOIN (
    SELECT 
        car_id,
        MAX(reported_at) AS max_reported_at
    FROM reports
    GROUP BY car_id
) latest ON r.car_id = latest.car_id AND r.reported_at = latest.max_reported_at
ORDER BY r.reported_at DESC;
```

### Explanation
This query identifies the most recent status for each vehicle in the system, which is critical for understanding current risk exposure and preventing duplicate investigations. The subquery first finds the latest report date for each car_id using MAX aggregation, then the main query joins this back to the reports table to retrieve the full record matching that maximum date. This approach handles the common scenario where a vehicle may have multiple reports over time. The results are ordered by reported date descending to show the most recently affected vehicles first. The output reveals 105 unique vehicles with reports, with three instances of plate "RHB-0442" appearing multiple times, suggesting a repeat-theft target that warrants additional security measures.

---

## Question 7: Average distance from owner by report status

### SQL Query
```sql
%%sql q7_result <<
SELECT 
    status,
    ROUND(AVG(distance_from_owner_km), 2) AS avg_distance_km,
    COUNT(*) AS reports_count
FROM reports
GROUP BY status
ORDER BY avg_distance_km DESC;
```

### Explanation
This query examines whether the distance a vehicle is found from its owner's location correlates with case outcome, providing insight into geographic dispersion patterns across different case types. By grouping reports by status and calculating the average distance, we can identify whether certain outcomes (like "Found" cases) tend to occur closer to or farther from the owner's location. The ROUND function ensures readability with 2 decimal places, and including reports_count provides context for the reliability of each average. Results show remarkably similar average distances across all statuses (140-149 km), suggesting that distance alone may not be a strong predictor of case outcome. This finding implies that other factors like investigator response time or vehicle security features may be more influential.

---

## Question 8: Owners with cars reported more than once

### SQL Query
```sql
%%sql q8_result <<
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
ORDER BY total_reports DESC, owner_id ASC;
```

### Explanation
This query identifies repeat victims—individuals whose vehicles have been reported stolen or abandoned multiple times—representing a high-priority group for targeted intervention and security improvement programs. The query chains joins from individuals through cars to reports, counting the total number of reports associated with each owner's vehicles. The HAVING clause filters for owners with more than one report total across all their vehicles. This reveals 39 individuals in this category, with Kevin Flores experiencing 8 separate incidents. Such patterns suggest either inadequate security practices, geographic risk factors, or potentially organized targeting. These owners should receive priority assistance with vehicle tracking systems, parking security guidance, and insurance support.

---

## Question 9: Top 3 most reported cars per last-seen location

### SQL Query
```sql
%%sql q9_result <<
WITH ranked_reports AS (
    SELECT 
        l.name AS location,
        c.plate,
        COUNT(r.id) AS reports_count,
        ROW_NUMBER() OVER (PARTITION BY l.name ORDER BY COUNT(r.id) DESC, c.plate ASC) AS rn
    FROM reports r
    INNER JOIN cars c ON r.car_id = c.id
    INNER JOIN locations l ON r.last_seen_location_id = l.id
    GROUP BY l.name, c.plate
)
SELECT 
    location,
    plate,
    reports_count
FROM ranked_reports
WHERE rn <= 3
ORDER BY location ASC, reports_count DESC, plate ASC;
```

### Explanation
This query identifies geographic hotspots by finding the most frequently reported vehicles at each location, using a Common Table Expression (CTE) with window functions for sophisticated ranking logic. The CTE first groups reports by location and vehicle plate, counting occurrences, then applies ROW_NUMBER() to rank vehicles within each location based on report count. The PARTITION BY clause ensures ranking resets for each location, while the ORDER BY within OVER prioritizes higher counts and uses plate as a tiebreaker for consistency. The main query filters for only the top 3 ranks per location. Results reveal that "Abuja Zone 11" has a vehicle (RHB-0442) with 2 reports, the highest single-vehicle concentration at any location, suggesting it may be a repeat target or a high-traffic area for vehicle crime.

---

## Question 10: First (earliest) report assigned to each investigator by jurisdiction

### SQL Query
```sql
%%sql q10_result <<
WITH investigator_earliest AS (
    SELECT 
        inv.id AS investigator_id,
        inv.first_name || ' ' || inv.last_name AS investigator,
        r.id AS report_id,
        r.reported_at,
        l.name AS jurisdiction,
        ROW_NUMBER() OVER (PARTITION BY inv.id ORDER BY r.reported_at ASC) AS rn
    FROM investigators inv
    INNER JOIN reports r ON inv.id = r.investigator_id
    INNER JOIN locations l ON inv.jurisdiction_location_id = l.id
)
SELECT 
    investigator_id,
    investigator,
    report_id,
    reported_at,
    jurisdiction
FROM investigator_earliest
WHERE rn = 1
ORDER BY investigator_id ASC;
```

### Explanation
This query tracks each investigator's first assigned case, which is valuable for analyzing response patterns, training effectiveness, and initial case assignment protocols. Using a CTE with ROW_NUMBER() window function, we rank each investigator's reports chronologically, partitioning by investigator_id to ensure each gets their own sequence. The ORDER BY reported_at ASC within the window function prioritizes the earliest case. The main query then filters for rank 1 (rn = 1) to select only the first report per investigator. Results show 47 investigators with at least one assigned case, with Andrew Lee's first case dating to May 2024, while others like Edward Davis started in September 2024. This temporal spread might indicate different hiring cohorts or varying jurisdictional workloads that required investigator assignment at different times.

---

## Strategic Analysis Section

### Prioritization: Most Important Insight

**Selected Insight: Question 8 - Repeat Victim Analysis**

Of all the analyses conducted, I believe the repeat victim identification (Question 8) provides the most actionable and high-impact insight for stakeholders. This query reveals 39 individuals who have experienced multiple vehicle theft/loss incidents, with the top victim suffering 8 separate occurrences. 

**Why This Matters:**
1. **Immediate Intervention Opportunity** - These 39 individuals represent concentrated risk and immediate candidates for targeted prevention programs
2. **Resource Efficiency** - Helping 39 repeat victims could reduce future case volume more effectively than broad awareness campaigns
3. **Pattern Detection** - Multiple incidents per person suggest systematic vulnerabilities (inadequate security, risky parking locations, or targeted theft rings)
4. **Economic Impact** - Repeat victims face compounding financial losses, insurance complications, and loss of mobility
5. **Prevention Focus** - Addressing root causes for this group shifts resources from reactive investigation to proactive prevention

**Recommended Action:**
Establish a "High-Risk Vehicle Owner" program that proactively contacts these 39 individuals with:
- Free vehicle tracking system installation
- Personalized security consultation based on their location patterns
- Direct liaison with insurance partners for improved coverage
- Priority notification systems for their zones
- Investigation into whether these are targeted thefts or security gaps

This targeted intervention could potentially prevent 50+ future incidents if each repeat victim experiences even half of their previous incident rate.

---

### Data Limitations

**Key Limitation: Selection Bias in Reporting**

The most significant limitation is that this dataset only captures *reported* vehicle crimes. Many vehicle thefts and losses likely go unreported due to:
- Lack of insurance (no incentive to report)
- Low confidence in recovery (perceived futility)
- Informal resolutions (vehicles found before formal reporting)
- Rural area access barriers (difficulty reaching reporting centers)
- Documentation gaps (owners lacking vehicle registration proof)

**Decision Impact:**
This limitation creates several analytical blind spots:
- **Geographic Patterns** - Zones with lower reported crime may simply have lower reporting rates, not lower actual crime rates
- **Demographic Bias** - Higher-income vehicle owners may be overrepresented due to insurance requirements
- **Severity Distortion** - We're missing data on the most vulnerable owners who can't afford to report
- **Prevention Effectiveness** - We cannot measure impact on unreported incidents

**Mitigation Strategies:**
To improve data quality, stakeholders should:
- Cross-reference insurance claims data for underreporting estimation
- Conduct community surveys in zones with suspiciously low report counts
- Implement simplified mobile/SMS reporting to reduce access barriers
- Partner with informal mechanics who might see unreported stolen vehicles

---

### Query Scalability Awareness

**Current State Analysis:**
With 150 reports and 200 vehicles, current queries execute instantly. However, if this dataset scaled 10x to 1,500 reports and 2,000 vehicles, several queries would require optimization:

**Questions Requiring Modification:**

1. **Question 6 (Latest Report per Car)** - The self-join approach would become expensive
   - **Solution:** Create an indexed materialized view or use window functions instead:
   ```sql
   WITH ranked AS (
       SELECT *, ROW_NUMBER() OVER (PARTITION BY car_id ORDER BY reported_at DESC) AS rn
       FROM reports
   )
   SELECT * FROM ranked WHERE rn = 1;
   ```

2. **Question 9 (Top 3 per Location)** - Window function over large grouped data could slow
   - **Solution:** Add indexes on `last_seen_location_id` and `car_id` in reports table
   - Consider pre-aggregating location statistics in a summary table updated nightly

3. **Question 3 (Recent Reports with 4-table JOIN)** - Multiple joins would be costly
   - **Solution:** Index foreign keys (car_id, individual_id, last_seen_location_id)
   - Consider a denormalized reporting view that pre-joins commonly accessed fields

**General Scalability Principles Applied:**
- Add appropriate indexes on all foreign key columns
- Use EXPLAIN QUERY PLAN to identify full table scans
- Partition reports table by date ranges if data spans years
- Implement incremental aggregation for questions 4, 5, 7 (update statistics rather than recalculate)
- Move window function logic to application layer for questions 9 and 10 when result sets exceed 10,000 rows

**No Immediate Changes Needed:**
Questions 1, 2, 4, 7, and 8 use efficient GROUP BY aggregations that scale well with proper indexing.

---

## Final Capstone Task: Original Analytical Insight

### Research Question
**"Does geographic clustering of incidents suggest organized theft rings versus opportunistic crime?"**

### SQL Query
```sql
%%sql capstone_result <<
WITH location_patterns AS (
    SELECT 
        l.name AS location,
        l.city,
        COUNT(DISTINCT r.id) AS total_incidents,
        COUNT(DISTINCT r.car_id) AS unique_vehicles,
        COUNT(DISTINCT c.individual_id) AS unique_owners,
        ROUND(AVG(r.distance_from_owner_km), 1) AS avg_distance_km,
        COUNT(CASE WHEN r.status = 'Abandoned' THEN 1 END) AS abandoned_count,
        ROUND(
            COUNT(DISTINCT r.id) * 1.0 / 
            COUNT(DISTINCT c.individual_id), 
            2
        ) AS incidents_per_owner
    FROM reports r
    INNER JOIN cars c ON r.car_id = c.id
    INNER JOIN locations l ON r.last_seen_location_id = l.id
    GROUP BY l.id, l.name, l.city
    HAVING COUNT(DISTINCT r.id) >= 3
)
SELECT 
    location,
    city,
    total_incidents,
    unique_vehicles,
    unique_owners,
    avg_distance_km,
    abandoned_count,
    incidents_per_owner,
    CASE 
        WHEN incidents_per_owner >= 1.5 THEN 'Possible Repeat Targeting'
        WHEN abandoned_count >= total_incidents * 0.6 THEN 'Possible Chop Shop Zone'
        WHEN avg_distance_km > 200 THEN 'Long-Distance Transit Hub'
        ELSE 'Standard Opportunistic Crime'
    END AS pattern_classification
FROM location_patterns
ORDER BY total_incidents DESC, incidents_per_owner DESC;
```

### Analytical Insight

**What the Data Shows:**
The query reveals distinct geographic patterns in vehicle crime across Nigerian police zones. Key findings include:

1. **Enugu Zone 12** leads with 7 incidents affecting 7 vehicles but only 5 unique owners (1.40 incidents per owner), suggesting potential repeat targeting
2. **Port Harcourt Zone 4, 13, and 17** show high incident rates with diverse vehicle types, indicating possible organized operations
3. **Abuja Zone 1, 3, and 11** demonstrate concentrated activity with moderate distances (86-149 km), suggesting local opportunistic theft
4. **Abandoned vehicle rates** vary dramatically by zone (33-100%), potentially indicating different crime motivations

**Why This Matters:**
Understanding whether crime is opportunistic or organized fundamentally changes enforcement strategy:
- **Organized Rings** require intelligence-led operations, surveillance, and cross-jurisdictional coordination
- **Opportunistic Crime** responds better to environmental design improvements (lighting, security patrols) and owner education
- **Transit Hubs** (high distance averages) need interstate cooperation and tracking technology

The incidents_per_owner ratio is particularly revealing: values above 1.5 suggest criminals are specifically targeting certain individuals or vehicle types rather than randomly selecting opportunities. This warrants immediate investigation into whether these are insurance fraud schemes, targeted business competitors, or theft rings with specific buyer demands.

### Practical Recommendation

**Three-Tiered Geographic Response Strategy:**

**Tier 1 - High-Risk Organized Crime Zones** (incidents_per_owner > 1.5)
- Deploy undercover operations in Enugu Zone 12 and similar areas
- Install automated license plate readers at zone boundaries
- Coordinate with neighboring jurisdictions on cross-border tracking

**Tier 2 - Transit/Chop Shop Zones** (high abandonment rate + high distance)
- Increase patrols at transportation hubs and border crossings
- Partner with scrap metal dealers and mechanics for stolen vehicle identification
- Implement mandatory vehicle documentation checks at repair shops

**Tier 3 - Opportunistic Crime Zones** (standard classification)
- Focus on community awareness campaigns and parking security
- Improve street lighting and CCTV coverage
- Offer subsidized vehicle tracking systems to high-value vehicle owners

This geographic segmentation allows the Nigerian Police Force to allocate limited investigative resources based on crime sophistication level rather than merely responding to incident volume.

---

## Ethical and Social Considerations

**Critical Consideration: Bias in Geographic Enforcement**

When interpreting and presenting these results, we must be extremely cautious about reinforcing discriminatory enforcement patterns or stigmatizing communities based on crime data.

**Key Ethical Concerns:**

1. **Selection Bias** - Zones with higher reported crime may reflect better reporting infrastructure or trust in police, not necessarily higher actual crime rates. Presenting "high-crime zones" without this context could misdirect resources away from areas with poor reporting access.

2. **Socioeconomic Stigma** - Publicly identifying neighborhoods as "high-risk" can decrease property values, reduce business investment, and create self-fulfilling prophecies where residents face discrimination in insurance, employment, and lending.

3. **Racial and Ethnic Profiling** - If certain geographic zones correlate with ethnic or religious demographics, increased enforcement could unfairly target specific communities. Data-driven policing must not become algorithmic discrimination.

4. **Privacy of Repeat Victims** - Individuals with multiple reports (Question 8) are vulnerable. Publicizing repeat victim status could expose them to further targeting or suggest victim-blaming narratives (e.g., "They must be careless").

5. **Resource Equity** - Allocating more investigators to "high-crime" zones may perpetuate disparities if those zones are under-resourced in other public services (education, employment, infrastructure).

**Responsible Practices:**

- Present findings with explicit acknowledgment of data limitations and potential biases
- Avoid labeling entire communities as "criminal hotspots"; focus on specific patterns
- Ensure community representatives participate in interpreting results and designing interventions
- Implement parallel investment in social services and economic opportunity in high-incident areas
- Regularly audit whether enforcement increases correlate with actual crime reduction or just arrest rate inflation
- Protect individual privacy by aggregating data and never publicly identifying repeat victims without consent

**Fundamental Principle:**
This analysis should inform prevention and victim support strategies, not justify heavy-handed enforcement or community surveillance that erodes trust and civil liberties. The goal is safer communities through partnership, not criminalization of geography.

---

## Follow-Up Analysis Recommendations

**If I had more time or additional data, I would explore:**

### 1. Temporal Pattern Analysis
**Question:** Are there seasonal, day-of-week, or time-of-day patterns in vehicle crimes?
**Data Needed:** Hourly timestamp data, calendar integration for holidays/events
**Why It Matters:** Would enable predictive patrol scheduling and targeted prevention campaigns during high-risk periods
**Approach:** Time-series analysis with seasonality decomposition, heatmaps of incident frequency by hour/day

### 2. Vehicle Make/Model Risk Profiling
**Question:** Which vehicle makes, models, and years are disproportionately targeted?
**Data Needed:** Complete vehicle population data (not just stolen vehicles) to calculate risk rates
**Why It Matters:** Could inform owner warnings and insurance risk assessment models
**Approach:** Calculate theft rate per 1000 vehicles for each make/model, control for market share

### 3. Investigation Outcome Effectiveness Analysis
**Question:** What factors correlate with successful vehicle recovery (status = 'Found')?
**Data Needed:** Case resolution timeline, initial investigator response time, vehicle tracking technology presence
**Why It Matters:** Would identify best practices for case handling and justify resource investments
**Approach:** Logistic regression predicting 'Found' status based on response time, distance, investigator workload, vehicle attributes

### 4. Social Network Analysis of Repeat Victims
**Question:** Are repeat victims connected through location, employer, or social networks?
**Data Needed:** Home addresses, workplaces, social media connections (with consent)
**Why It Matters:** Could reveal whether organized theft rings are targeting specific communities or businesses
**Approach:** Graph analysis to identify clusters, common addresses/locations visited

### 5. Economic Impact Assessment
**Question:** What is the total economic cost of vehicle crime per zone, considering vehicle value, loss of mobility, and investigation costs?
**Data Needed:** Vehicle values, owner income/occupation, investigation man-hours, insurance payout data
**Why It Matters:** Would justify budget allocation for prevention programs and quantify community burden
**Approach:** Cost-benefit analysis comparing prevention program investment versus projected loss reduction

**Highest Priority Follow-Up:**
I would prioritize **Analysis #3 (Investigation Effectiveness)** because it directly addresses operational improvement with available data. Understanding what predicts successful recovery would immediately improve case management protocols and victim outcomes without requiring extensive new data collection.

---

## Conclusion

This portfolio project demonstrates core data analyst competencies: writing accurate SQL queries, explaining technical work to non-technical audiences, reasoning about data limitations, and providing actionable recommendations. The analysis reveals that vehicle crime in the CMR database shows both opportunistic and organized patterns, with significant opportunities for targeted prevention programs focused on repeat victims and high-risk geographic zones.

The work showcases SQL proficiency across fundamental techniques (joins, aggregations) and advanced methods (window functions, CTEs) while maintaining awareness of ethical implications and scalability considerations. Most importantly, it demonstrates the ability to move from data exploration to strategic insight—a critical skill for any data analyst supporting operational teams.

**Key Competencies Validated:**
✅ SQL query construction and optimization  
✅ Relational database navigation and analysis  
✅ Statistical reasoning and pattern recognition  
✅ Stakeholder communication and business impact framing  
✅ Ethical data interpretation and bias awareness  
✅ Forward-thinking scalability considerations  
