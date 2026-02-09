# AML Transaction Monitoring System

## Overview

The AML (Anti-Money Laundering) Transaction Monitoring System is a compliance solution designed to identify, track, and report suspicious financial transactions. The system processes transactional data, applies predefined detection rules, and generates comprehensive alerts for compliance review through an interactive Excel-based dashboard.

## Features

### 🎯 Core Capabilities
- **Automated Alert Generation**: Apply multiple detection rules simultaneously to identify suspicious activities
- **Real-time Monitoring**: Process transactions and generate alerts within 24 hours
- **Comprehensive Dashboard**: Interactive Excel-based visualization with KPIs, charts, and drill-down capabilities
- **Risk-Based Classification**: Categorize alerts by risk level (High/Medium) for prioritized investigation
- **Trend Analysis**: Track suspicious activity patterns over time with monthly trend charts

### 📊 Dashboard Components

#### 1. **Key Performance Indicators (KPIs)**
- Total Alerts Count
- High Risk Alerts
- High-Risk Country Transactions
- Large Transactions (>$40K)

#### 2. **Analytics & Visualizations**
- **Alerts by Rule**: Bar chart showing distribution across detection rules
- **Alerts by Country**: Top 10 countries ranked by alert volume
- **Monthly Trend**: Line chart tracking suspicious transaction patterns
- **Top Risky Customers**: Ranked list of customers with most alerts

#### 3. **Data Sheets**
- **Dashboard**: Executive summary with all KPIs and charts
- **Alerts Data**: Complete alert list with filtering capabilities
- **Summary Data**: Pre-aggregated data for pivot analysis
- **Instructions**: User guide and system documentation

## Detection Rules

The system implements four core detection rules:

| Rule ID | Rule Name | Criteria | Risk Level |
|---------|-----------|----------|------------|
| R001 | High-Risk Country | Transactions from Iran, Syria, North Korea, Afghanistan | High |
| R002 | Large Deposit | Deposits > $10,000 | Medium |
| R003 | Large Transfer | Transfers > $5,000 | Medium |
| R004 | Very Large Amount | Any transaction > $40,000 | High |

## Data Schema

### Transaction Table

```sql
CREATE TABLE transactions (
    txn_id VARCHAR(10),
    customer_id VARCHAR(10),
    txn_date DATE,
    amount DECIMAL(12,2),
    txn_type VARCHAR(50),
    country VARCHAR(50),
    channel VARCHAR(50)
);
```

### Alert Table Structure

| Field | Type | Description |
|-------|------|-------------|
| Alert_ID | Integer | Unique alert identifier |
| Txn_ID | VARCHAR(10) | Reference to transaction |
| Customer_ID | VARCHAR(10) | Customer identifier |
| Txn_Date | DATE | Transaction date |
| Amount | DECIMAL(12,2) | Transaction amount (USD) |
| Txn_Type | VARCHAR(50) | Purchase, Withdrawal, Deposit, Transfer, Refund, Payment |
| Country | VARCHAR(50) | Transaction origin country |
| Channel | VARCHAR(50) | Online, Mobile App, ATM, Branch, Phone Banking, POS |
| Alert_Rule | VARCHAR(50) | Detection rule that triggered the alert |
| Risk_Level | VARCHAR(10) | High or Medium |

## Installation & Setup

### Prerequisites
- Microsoft Excel 2016 or later (for full dashboard functionality)
- LibreOffice Calc (alternative, with limited chart support)
- Database system (PostgreSQL, MySQL, or SQL Server)

### Quick Start

1. **Load Transaction Data**
   ```sql
   -- Import transactions.sql into your database
   psql -U username -d database_name -f transactions.sql
   ```

2. **Generate Alerts**
   ```sql
   -- Run alert generation queries
   -- See 'SQL Queries' section below
   ```

3. **Open Dashboard**
   - Open `aml_dashboard.xlsx` in Microsoft Excel
   - Enable macros if prompted (for full functionality)
   - Review the Instructions sheet for detailed usage guide

## Usage Guide

### For Compliance Officers

#### Daily Workflow
1. Open the Dashboard sheet for high-level overview
2. Review KPI metrics for any unusual spikes
3. Check Monthly Trend chart for pattern changes
4. Navigate to Alerts Data sheet
5. Filter by Risk_Level = "High" for priority review
6. Investigate flagged transactions
7. Document findings in your case management system

#### Investigation Process
1. **Identify Alert**: Review alert details (amount, country, customer)
2. **Customer Profile**: Cross-reference Customer_ID for account history
3. **Pattern Analysis**: Filter all alerts for same customer
4. **Country Risk**: Check if country is on sanctions list
5. **Documentation**: Record investigation notes
6. **Disposition**: Escalate to SAR filing or close as false positive

### Using Filters and Slicers

#### Apply Column Filters
1. Click any header in Alerts Data table
2. Use dropdown to filter by specific values
3. Combine multiple filters for advanced analysis

#### Add Slicers (Excel 2016+)
1. Click anywhere in Alerts Data table
2. Go to Table Design → Insert Slicer
3. Select fields: Country, Alert_Rule, Risk_Level, Channel
4. Click multiple values to filter data dynamically

### Example Queries

#### Find All High-Risk Country Transactions
```sql
SELECT * 
FROM transactions
WHERE country IN ('Iran', 'Syria', 'North Korea', 'Afghanistan');
```

#### Identify Large Deposits
```sql
SELECT customer_id, txn_date, SUM(amount) as total_amount
FROM transactions
WHERE txn_type = 'Deposit'
GROUP BY customer_id, txn_date
HAVING SUM(amount) > 10000;
```

#### Top Risky Customers
```sql
SELECT customer_id, 
       COUNT(*) as alert_count,
       SUM(CASE WHEN amount > 40000 THEN 1 ELSE 0 END) as high_value_txns,
       SUM(amount) as total_amount
FROM transactions
WHERE country IN ('Iran', 'Syria', 'North Korea', 'Afghanistan')
   OR amount > 10000
GROUP BY customer_id
ORDER BY alert_count DESC
LIMIT 20;
```

#### Monthly Suspicious Transaction Trend
```sql
SELECT DATE_TRUNC('month', txn_date) as month,
       COUNT(*) as alert_count,
       SUM(amount) as total_amount
FROM transactions
WHERE amount > 10000 
   OR country IN ('Iran', 'Syria', 'North Korea', 'Afghanistan')
GROUP BY DATE_TRUNC('month', txn_date)
ORDER BY month;
```

## File Structure

```
aml-transaction-monitoring/
├── aml_dashboard.xlsx              # Main dashboard file
├── transaction_data.xlsx           # Sample transaction dataset (550 rows)
├── transactions.sql                # SQL file with all transactions
├── query_result.xlsx               # Large deposits query results
├── grouped_query_result.xlsx       # Grouped deposit analysis
├── high_risk_countries_transactions.xlsx  # High-risk country alerts
├── transfer_over_5000.xlsx         # Large transfer alerts
├── AML_Transaction_Monitoring_BRD.docx    # Business Requirements Document
└── README.md                       # This file
```

## Dashboard Metrics Explained

### Total Alerts
Count of all alerts generated across all detection rules. A single transaction may trigger multiple rules.

### High Risk Alerts
Count of alerts classified as "High" severity. Currently includes:
- High-Risk Country transactions
- Very Large Amount transactions (>$40K)

### High-Risk Countries
Total number of alerts involving transactions from sanctioned countries (Iran, Syria, North Korea, Afghanistan).

### Large Transactions
Count of transactions exceeding $40,000, regardless of type or country.

## Regulatory Compliance

### Supported Regulations
- **Bank Secrecy Act (BSA)**: Transaction monitoring and SAR filing requirements
- **USA PATRIOT Act**: Enhanced due diligence for high-risk jurisdictions
- **OFAC Sanctions**: Screening against sanctioned countries
- **FATF Recommendations**: Risk-based approach to AML/CFT

### Reporting Requirements
- **Suspicious Activity Reports (SARs)**: File within 30 days of detection
- **Currency Transaction Reports (CTRs)**: Auto-flagged for >$10K cash transactions
- **Enhanced Due Diligence (EDD)**: Required for all high-risk country alerts

## Customization

### Adding New Detection Rules

1. **Define Rule Logic**
   ```sql
   -- Example: Rapid succession deposits
   SELECT customer_id, COUNT(*) as deposit_count
   FROM transactions
   WHERE txn_type = 'Deposit'
     AND txn_date >= CURRENT_DATE - INTERVAL '7 days'
   GROUP BY customer_id
   HAVING COUNT(*) > 5;
   ```

2. **Update Dashboard**
   - Add new row to "Alerts by Rule" section
   - Update formula: `=COUNTIF('Alerts Data'!I:I,"New Rule Name")`
   - Refresh charts to include new rule

### Modifying Thresholds

Edit the SQL queries or Excel formulas:

```sql
-- Change large deposit threshold from $10,000 to $15,000
WHERE txn_type = 'Deposit' AND amount > 15000
```

### Adding Countries to High-Risk List

Update the country list in queries:
```sql
WHERE country IN ('Iran', 'Syria', 'North Korea', 'Afghanistan', 'NewCountry')
```

And update the dashboard formula:
```excel
=COUNTIF('Alerts Data'!G:G,"NewCountry") + ...
```

## Performance Optimization

### For Large Datasets (>100,000 transactions)

1. **Database Indexing**
   ```sql
   CREATE INDEX idx_amount ON transactions(amount);
   CREATE INDEX idx_country ON transactions(country);
   CREATE INDEX idx_txn_date ON transactions(txn_date);
   CREATE INDEX idx_customer ON transactions(customer_id);
   ```

2. **Partitioning**
   ```sql
   -- Partition by month for faster queries
   CREATE TABLE transactions_partitioned (
       LIKE transactions
   ) PARTITION BY RANGE (txn_date);
   ```

3. **Dashboard Optimization**
   - Limit Alerts Data to last 90 days for performance
   - Archive historical alerts to separate workbook
   - Use Excel's Data Model for large datasets

## Troubleshooting

### Common Issues

#### Dashboard Shows #REF! Errors
**Cause**: Broken cell references
**Solution**: Re-download the dashboard file or check formula references

#### Charts Not Updating
**Cause**: Manual calculation mode enabled
**Solution**: Press `Ctrl + Alt + F9` to force recalculation, or set Calculation to Automatic in Excel Options

#### Slow Performance
**Cause**: Too many rows in Alerts Data sheet
**Solution**: 
- Filter to show only recent alerts (last 30-90 days)
- Archive older alerts to separate file
- Remove unused slicers

#### Missing Data After Filtering
**Cause**: Multiple filters applied simultaneously
**Solution**: Clear all filters (Data → Clear) and reapply selectively

## Best Practices

### Daily Operations
✅ Review dashboard daily before 10 AM
✅ Prioritize High risk alerts first
✅ Document all investigation findings
✅ Update alert dispositions within 5 business days
✅ Escalate unusual patterns to management

### Data Quality
✅ Validate transaction data completeness daily
✅ Check for duplicate transactions
✅ Verify country codes match ISO standards
✅ Ensure date formats are consistent

### Security
✅ Protect Excel file with password
✅ Store in secure, encrypted location
✅ Limit access to authorized personnel only
✅ Maintain audit log of dashboard access
✅ Backup dashboard weekly

## Maintenance

### Weekly Tasks
- Review false positive rate
- Update high-risk country list (check OFAC)
- Archive alerts older than 90 days
- Backup dashboard and data files

### Monthly Tasks
- Analyze alert trends and patterns
- Tune detection rule thresholds
- Generate management reports
- Review user access permissions

### Quarterly Tasks
- Comprehensive rule effectiveness review
- Update BRD with any changes
- User training refresher sessions
- Regulatory requirement validation

## Support & Documentation

### Additional Resources
- **Business Requirements Document**: `AML_Transaction_Monitoring_BRD.docx`
- **Instructions Sheet**: Inside `aml_dashboard.xlsx`
- **SQL Reference**: All sample queries included in this README

### Training Materials
- Dashboard walkthrough video: [Link to be added]
- Investigation procedures guide: See BRD Section 6
- Compliance team contacts: [To be populated]

## Version History

### Version 1.0 (February 2026)
- Initial release
- 4 core detection rules implemented
- Excel dashboard with 4 worksheets
- Sample dataset with 550 transactions
- 383 alerts generated from sample data
- Complete BRD documentation

### Planned Enhancements (v1.1)
- [ ] Automated daily refresh from database
- [ ] Email notifications for high-risk alerts
- [ ] Machine learning-based anomaly detection
- [ ] Integration with case management system
- [ ] Mobile dashboard view

## Contributing

### Reporting Issues
If you encounter bugs or have feature requests:
1. Document the issue with screenshots
2. Include steps to reproduce
3. Note your Excel version and OS
4. Submit to compliance team lead

### Suggesting Improvements
We welcome suggestions for:
- New detection rules
- Dashboard enhancements
- Process improvements
- Documentation updates

## License

Proprietary - Internal Use Only
© 2026 Compliance & Risk Management Department
All rights reserved.

---

## Quick Reference Card

### Most Common Tasks

| Task | Action |
|------|--------|
| View all high-risk alerts | Filter Risk_Level = "High" |
| Find customer's alerts | Filter Customer_ID = "CUSTXXXXX" |
| Check monthly trends | View Dashboard → Monthly Trend chart |
| Export filtered data | Select filtered rows → Copy → Paste to new file |
| Add slicer | Click table → Table Design → Insert Slicer |
| Refresh calculations | Press F9 or Ctrl+Alt+F9 |

### Key Formulas

```excel
Total Alerts:        =COUNTA('Alerts Data'!A2:A10000)
High Risk Count:     =COUNTIF('Alerts Data'!J:J,"High")
Country Alert Count: =COUNTIF('Alerts Data'!G:G,"CountryName")
```

### Contact Information

**Compliance Team**: compliance@organization.com
**Technical Support**: it-support@organization.com
**Emergency Escalation**: [Phone number]

---

*Last Updated: February 09, 2026*
*Document Version: 1.0*
