# Taxi Rides NY - dbt Project

A comprehensive dbt project for analyzing New York City taxi ride data, built with a modern data warehouse architecture.

## 📊 Project Overview

This project processes and analyzes NYC taxi trip data from both green and yellow taxi services. It transforms raw data into structured, analytics-ready tables suitable for business intelligence, reporting, and data analysis.

### Key Features

- **Multi-source data processing**: Handles both green and yellow taxi tripdata
- **Layered architecture**: Staging → Intermediate → Mart → Reporting
- **Comprehensive data quality testing**: Extensive test suite at each layer
- **Business-ready models**: Fact and dimension tables optimized for analytics
- **Monthly reporting**: Aggregated revenue analysis by zone and service type

## 🏗️ Architecture

### Data Flow

```
Raw Data → Staging → Intermediate → Mart → Reporting
    ↓          ↓          ↓          ↓          ↓
  Green/     Cleaned    Enriched   Dimensional Business
  Yellow     & Validated & Dedup.   Models     Reports
  Tripdata                Data
```

### Project Structure

```
taxi_rides_ny/
├── models/
│   ├── staging/              # Raw data cleaning and validation
│   │   ├── stg_green_tripdata.sql
│   │   ├── stg_yellow_tripdata.sql
│   │   └── sources.yml
│   ├── intermediate/         # Data enrichment and transformation
│   │   ├── int_trips_unioned.sql
│   │   ├── int_trips.sql
│   │   └── schema.yml
│   ├── marts/               # Dimensional models
│   │   ├── dim_zones.sql
│   │   ├── dim_vendors.sql
│   │   ├── fct_trips.sql
│   │   └── schema.yml
│   └── reporting/           # Business reporting models
│       ├── fct_monthly_zone_reporting.sql
│       └── schema.yml
├── macros/                  # Reusable SQL macros
│   ├── get_trip_duration_minutes.sql
│   ├── get_vendor_data.sql
│   └── soft_cast.sql
├── seeds/                  # Reference data
│   ├── payment_type_lookup.csv
│   └── taxi_zone_lookup.csv
├── tests/                  # Comprehensive test suite
│   ├── generic/            # Reusable test macros
│   ├── README.md           # Test documentation
│   ├── staging_tests.md    # Staging layer tests
│   └── reporting_tests.md # Reporting layer tests
└── dbt_project.yml         # Project configuration
```

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- dbt Core installed
- Access to a supported database (PostgreSQL, BigQuery, Snowflake, etc.)
- NYC taxi tripdata

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd taxi_rides_ny
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Set up your profiles:
```bash
# Create ~/.dbt/profiles.yml with your database connection
```

### Configuration

Update `dbt_project.yml` to match your environment:

```yaml
name: 'taxi_rides_ny'
version: '1.0.0'
profile: 'taxi_rides_ny'

# Configure your target database
model-paths: ["models"]
target-path: "target"
```

## 📊 Data Models

### Staging Layer

**Purpose**: Clean and validate raw data

- `stg_green_tripdata`: Raw green taxi data with basic validation
- `stg_yellow_tripdata`: Raw yellow taxi data with basic validation

**Key Operations**:
- Data type casting
- Null value filtering
- Basic validation
- Standardization of field names

### Intermediate Layer

**Purpose**: Enrich and deduplicate data

- `int_trips_unioned`: Combined green and yellow taxi data
- `int_trips`: Enriched with payment type descriptions and deduplicated

**Key Operations**:
- Data union and standardization
- Payment type enrichment
- Surrogate key generation
- Deduplication logic

### Mart Layer

**Purpose**: Create dimensional models for analytics

- `dim_zones`: NYC taxi zones with borough information
- `dim_vendors`: Taxi technology vendors
- `fct_trips`: Fact table with trip metrics and zone information

**Key Operations**:
- Dimension table creation
- Fact table aggregation
- Zone enrichment
- Time-based analysis

### Reporting Layer

**Purpose**: Business-ready aggregated models

- `fct_monthly_zone_reporting`: Monthly revenue analysis by zone and service type

**Key Operations**:
- Monthly aggregation
- Revenue breakdown
- Business metrics calculation

## 🧪 Testing

### Test Coverage

The project includes comprehensive tests at each layer:

#### Staging Tests
- Data integrity (not_null, unique)
- Data quality (positive_values, reasonable_*)
- Business logic validation
- Reference data validation

#### Intermediate Tests
- Relationship validation
- Data completeness checks
- Business rule enforcement

#### Mart Tests
- Dimensional integrity
- Relationship validation
- Data quality metrics

#### Reporting Tests
- Revenue validation
- Aggregation accuracy
- Business metric validation

### Running Tests

```bash
# Run all tests
dbt test

# Run tests for specific model
dbt test --select fct_trips

# Run tests by path
dbt test --select path:models/staging

# Run tests by tag
dbt test --select tag:data_quality
```

## 🔧 Macros

### Custom Macros

- `get_trip_duration_minutes()`: Calculate trip duration in minutes
- `get_vendor_data()`: Get vendor company names
- `soft_cast()`: Safe data type casting

### Usage Example

```sql
-- Calculate trip duration
select {{ get_trip_duration_minutes('pickup_datetime', 'dropoff_datetime') }} as duration_minutes

-- Get vendor information
select {{ get_vendor_data('vendor_id') }} as vendor_name
```

## 📈 Analytics & Business Intelligence

### Sample Queries

```sql
-- Daily trip counts by service type
select 
    service_type,
    date(pickup_datetime) as trip_date,
    count(*) as trip_count
from {{ ref('fct_trips') }}
group by 1, 2
order by 2 desc;

-- Monthly revenue by borough
select 
    pickup_borough,
    date_trunc('month', pickup_datetime) as revenue_month,
    sum(total_amount) as total_revenue,
    count(*) as trip_count
from {{ ref('fct_trips') }}
group by 1, 2
order by 2 desc;

-- Top pickup zones
select 
    pickup_zone,
    count(*) as trip_count,
    avg(fare_amount) as avg_fare
from {{ ref('fct_trips') }}
group by 1
order by 2 desc
limit 10;
```

### Business Metrics

- **Trip Volume**: Daily, weekly, monthly trip counts
- **Revenue Analysis**: Fare breakdown by service type and zone
- **Performance Metrics**: Average trip duration, distance, and fare
- **Geographic Analysis**: Popular pickup/dropoff zones and boroughs

## 🔍 Data Quality

### Quality Metrics

- **Data Completeness**: Null value monitoring
- **Data Accuracy**: Business rule validation
- **Data Consistency**: Cross-table relationship validation
- **Data Timeliness**: Freshness monitoring

### Quality Dashboard

The test suite provides insights into:
- Data volume trends
- Quality score tracking
- Issue identification and resolution
- Data lineage

## 🚀 Deployment

### Production Deployment

1. **Environment Setup**:
```bash
# Create production profile
cp profiles.yml.example profiles.yml
# Edit with production database credentials
```

2. **Run Production Build**:
```bash
dbt build --select state:modified
```

3. **Schedule Runs**:
```bash
# Set up cron job for daily updates
0 2 * * * cd /path/to/project && dbt run
```

### Monitoring

- Monitor dbt run logs
- Track test execution results
- Monitor data volume changes
- Set up alerts for test failures

## 🤝 Contributing

### Development Workflow

1. Create feature branch:
```bash
git checkout -b feature/new-analysis
```

2. Make changes and run tests:
```bash
dbt test
dbt build
```

3. Commit changes:
```bash
git commit -m "Add new analysis model"
```

4. Create pull request

### Code Standards

- Follow dbt best practices
- Include comprehensive tests
- Document new models
- Use meaningful variable names
- Keep models modular and reusable

## 📚 Documentation

### Additional Resources

- [dbt Documentation](https://docs.getdbt.com/)
- [NYC Taxi Data Documentation](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- [Data Warehouse Best Practices](https://docs.getdbt.com/guides/advanced/best-practices)

### Support

For issues and questions:
1. Check existing issues
2. Create new issue with detailed description
3. Include dbt run logs and error messages

## 📊 Performance Considerations

### Optimization Tips

- Use incremental models for large datasets
- Partition tables by date where appropriate
- Index frequently queried columns
- Monitor query performance

### Scalability

- Designed to handle millions of records
- Supports parallel processing
- Configurable for different database sizes
- Efficient data transformation patterns

## 🔄 Future Enhancements

### Planned Features

- Real-time data processing
- Advanced analytics models
- Machine learning integration
- Enhanced visualization dashboards
- Additional data sources (Uber, Lyft)

### Roadmap

1. **Q1 2024**: Real-time data pipeline
2. **Q2 2024**: Advanced analytics features
3. **Q3 2024**: Machine learning models
4. **Q4 2024**: Enhanced reporting suite

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👏 Acknowledgments

- NYC Taxi & Limousine Commission for providing the data
- dbt community for best practices and inspiration
- Contributors and maintainers of this project

---

**Built with ❤️ using dbt**