# VectorHub: Versatile Vector Search Analytics

## Overview

VectorHub is a powerful property search engine that combines semantic and geographic search capabilities. It uses vector embeddings to understand the meaning behind property descriptions and spatial indexing to find properties within specific geographic areas. The system provides a hybrid search algorithm that balances semantic relevance (70%) with geographic proximity (30%) to deliver highly relevant search results.

## Features

- **Hybrid Search**: Combines semantic similarity with geographic proximity
- **Vector Embeddings**: Uses Sentence Transformers to generate embeddings from property descriptions
- **Spatial Indexing**: Leverages MariaDB's spatial capabilities for geographic filtering
- **PDF Report Generation**: Creates professional PDF reports of search results
- **REST API**: Simple API for integration with other applications
- **Web Interface**: User-friendly interface for searching properties
- **Voice Search**: Supports speech-to-text for natural language queries
- **Text-to-Speech**: Can read out property descriptions

## Technology Stack

- **Backend**: Flask (Python)
- **Database**: MariaDB 11.7-rc with spatial indexing
- **Vector Search**: FAISS for efficient similarity search
- **Embeddings**: Sentence Transformers (all-MiniLM-L6-v2)
- **PDF Generation**: WeasyPrint
- **Frontend**: HTML/CSS/JavaScript (not included in this repository)

## Installation

### Prerequisites

- Python 3.8+
- MariaDB 11.7-rc or higher
- FAISS
- Required Python packages (see requirements.txt)

### Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/SakibAhmedShuva/VectorHub-Versatile-Vector-Search-Analytics.git
   cd VectorHub-Versatile-Vector-Search-Analytics
   ```

2. Create a virtual environment and install dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

3. Set up your environment variables in a `.env` file:
   ```
   DB_HOST=localhost
   DB_PORT=3306
   DB_USER=your_username
   DB_PASSWORD=your_password
   DB_NAME=vectorhub
   TABLE_NAME=embeddings
   SEARCH_QUERY="Modern house with pool"
   SEARCH_K=5
   BATCH_SIZE=32
   ```

4. Create the necessary directories:
   ```bash
   mkdir -p uploads results
   ```

## Usage

### Starting the Server

```bash
python app.py
```

The server will start on http://localhost:5000

### API Endpoints

#### Initialize System
```
POST /api/initialize
```
Upload a CSV file with property data to initialize the system.

Parameters:
- `file`: CSV file with property data (required)
- `clean_start`: Boolean to recreate tables (default: false)

#### Search Properties
```
POST /api/search
```
Search for properties based on text query and optional location.

Request body:
```json
{
  "query": "Modern house with pool",
  "latitude": 29.9668,
  "longitude": -95.6831,
  "k": 5,
  "radius_km": 10,
  "generate_pdf": false
}
```

#### Health Check
```
GET /health
```
Returns the health status of the application.

### Data Format

The CSV file should contain the following columns:
- `Latitude`: Property latitude (required)
- `Longitude`: Property longitude (required)
- Text columns for property descriptions:
  - `PrivateRemarks`
  - `PublicRemarks`
  - `BuildingFeatures`
  - `InteriorFeatures`
  - `ExteriorFeatures`
  - `ArchitecturalStyle`
  - `SpecialListingConditions`
  - `ZoningDescription`
  - `PoolFeatures`
  - `CommunityFeatures`
  - `FireplaceFeatures`

## How It Works

1. **Data Processing**: Property descriptions are processed and converted to vector embeddings using Sentence Transformers.
2. **Indexing**: Embeddings are stored in MariaDB and indexed with FAISS for fast retrieval.
3. **Hybrid Search**:
   - Semantic component (70%): Measures similarity between query and property descriptions using vector dot product.
   - Geographic component (30%): Calculates proximity based on the search location and radius.
4. **Result Ranking**: Properties are ranked by a combined score of semantic relevance and geographic proximity.

## Architecture

```
┌────────────┐     ┌────────────┐     ┌────────────┐
│   Client   │────▶│  Flask API │────▶│ SearchEngine│
└────────────┘     └────────────┘     └────────────┘
                          │                  │
                          ▼                  ▼
                   ┌────────────┐     ┌────────────┐
                   │ReportGenerator    │ FAISS Index │
                   └────────────┘     └────────────┘
                          │                  │
                          ▼                  ▼
                   ┌────────────┐     ┌────────────┐
                   │ WeasyPrint │     │ MariaDB    │
                   └────────────┘     └────────────┘
```

## Database Schema

The main table `embeddings` has the following structure:

```sql
CREATE TABLE embeddings (
    property_id INT PRIMARY KEY AUTO_INCREMENT,
    description TEXT NOT NULL,
    embedding JSON NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    location POINT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    SPATIAL INDEX(location),
    FULLTEXT INDEX(description)
)
```

## Performance Considerations

- The system uses FAISS for efficient similarity search in high-dimensional spaces
- Spatial indexing in MariaDB optimizes geographic filtering
- Batch processing for embedding generation improves throughput
- Combined scoring balances semantic and geographic relevance

## License

MIT License

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
