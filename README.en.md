[![id](https://img.shields.io/badge/lang-id-blue.svg)](https://github.com/mukhtarulll/archaeological-database/blob/main/README.md)

# Archaeological Research Management Database System
## Project: Database Design & Dashboard Visualization

This group project designs and implements a normalized relational database (MySQL) to manage archaeological research data, including expeditions, artifact discoveries, site locations, and laboratory results. An interactive dashboard built with Budibase provides real-time visualization of research activities and artifact analysis.

---

## Quick Summary

![Archaeological Dashboard](/Dokumentasi%20BudiBase/Dashboard%20Awal.png)

Dashboard displays:
- **Total 11 Artifacts** discovered by **5 Archaeologists** across **9 Sites**
- **Artifact Distribution by Culture** — Hindu-Buddha (18.2%), Sound Horeg (18.2%), Asdn (18.2%), etc
- **Top 5 Most Active Sites** — Situs Gunung Padang leads with 12 expeditions
- **Expedition Activity per Month** — Expedition trends July-Sept 2023 and June 2025
- **Latest Lab Results** — Morphology, composition, and dating analysis

---

## Database Architecture

### Entity-Relationship Diagram (ERD)
Database uses normalized **star schema** with 7 main entities:

![ERD](ERD%20Arkeolog.drawio.jpg)

### Main Tables

| Table | Primary Key | Description | Relationships |
|-------|-------------|-------------|---------------|
| `arkeolog` | id_arkeolog | Researcher data (name, institution, email) | 1:N → expedition |
| `situs_arkeologi` | id_situs | Site data (name, geographic location, period) | 1:N → location |
| `lokasi` | id_lokasi | Specific excavation grid at site | N:1 ← site, 1:N → artifact |
| `ekspedisi` | id_ekspedisi | Research activity (date, duration, archaeologist) | N:1 ← archaeologist, M:N ↔ artifact |
| `artefak` | id_artefak | Archaeological finding (type, dimensions, condition) | N:1 ← location, M:N ↔ expedition |
| `periode_budaya` | id_periode | Temporal classification (Hindu-Buddha, Megalithic, etc) | 1:N → artifact |
| `hasil_laboratorium` | id_hasil | Lab analysis results (dating, composition, morphology) | N:1 ← artifact |

---

## Database Features

### 1. Normalization (3NF)
- **Eliminates data redundancy** with foreign key relationships
- **Atomic values** in all columns
- **No transitive dependencies** — each non-key attribute depends only on primary key

### 2. Indexing
```sql
-- Performance indexes for queries
CREATE INDEX idx_situs_nama ON situs_arkeologi(nama_situs);
CREATE INDEX idx_artefak_periode ON artefak(id_periode);
CREATE INDEX idx_ekspedisi_tanggal ON ekspedisi(tanggal_mulai);
```

### 3. Referential Integrity
- Foreign key constraints to maintain data consistency
- CASCADE delete for junction table (artifact_expedition)
- RESTRICT delete for master data (sites, archaeologists)

---

## Analytical Queries

### Top 5 Most Productive Sites
```sql
SELECT 
    s.nama_situs,
    s.lokasi_geografis,
    COUNT(DISTINCT e.id_ekspedisi) as jumlah_ekspedisi,
    COUNT(DISTINCT a.id_artefak) as total_artefak
FROM situs_arkeologi s
LEFT JOIN lokasi l ON s.id_situs = l.id_situs
LEFT JOIN artefak a ON l.id_lokasi = a.id_lokasi
LEFT JOIN artefak_ekspedisi ae ON a.id_artefak = ae.id_artefak
LEFT JOIN ekspedisi e ON ae.id_ekspedisi = e.id_ekspedisi
GROUP BY s.id_situs, s.nama_situs, s.lokasi_geografis
ORDER BY jumlah_ekspedisi DESC, total_artefak DESC
LIMIT 5;
```

![Top 5 Sites](/Dokumentasi%20BudiBase/Top%205%20Situs%20AKtif.png)

### Artifact Distribution by Cultural Period
```sql
SELECT 
    pb.nama_periode,
    pb.rentang_tahun,
    COUNT(*) as jumlah_artefak,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM artefak), 1) as persentase
FROM periode_budaya pb
LEFT JOIN artefak a ON pb.id_periode = a.id_periode
GROUP BY pb.id_periode, pb.nama_periode, pb.rentang_tahun
ORDER BY jumlah_artefak DESC;
```

### Expedition Activity per Month
```sql
SELECT 
    DATE_FORMAT(tanggal_mulai, '%Y-%m') as bulan,
    COUNT(*) as jumlah_ekspedisi
FROM ekspedisi
GROUP BY DATE_FORMAT(tanggal_mulai, '%Y-%m')
ORDER BY bulan;
```

![Expedition Activity](/Dokumentasi%20BudiBase/Aktifitas%20Ekspedisi%20per%20Bulan.png)

### Most Productive Archaeologists
```sql
SELECT 
    ar.nama,
    ar.institusi,
    COUNT(e.id_ekspedisi) as jumlah_ekspedisi,
    COUNT(DISTINCT ae.id_artefak) as artefak_ditemukan
FROM arkeolog ar
LEFT JOIN ekspedisi e ON ar.id_arkeolog = e.id_arkeolog
LEFT JOIN artefak_ekspedisi ae ON e.id_ekspedisi = ae.id_ekspedisi
GROUP BY ar.id_arkeolog, ar.nama, ar.institusi
ORDER BY jumlah_ekspedisi DESC, artefak_ditemukan DESC;
```

![Most Productive Archaeologists](/Dokumentasi%20BudiBase/Arkeolog%20Terproduktif.png)

See `queries.sql` for complete and complex queries.

---

## Budibase Dashboard

### Dashboard Components
1. **KPI Cards**
   - Total Artifacts: 11
   - Total Archaeologists: 5 people
   - Total Sites: 9 locations

2. **Pie Chart: Artifact Distribution by Culture**
   - Visual proportion of artifacts per period (Hindu-Buddha, Sound Horeg, Megalithic, etc)

3. **Bar Chart: Expedition Activity per Month**
   - Temporal trend of research expeditions (spikes in July-Sept 2023 & June 2025)

4. **Horizontal Bar: Top 5 Most Active Sites**
   - Site ranking based on expedition count

5. **Data Table: Latest Lab Results**
   - List of recent lab analyses with dates, artifacts, results

6. **Gallery: Artifact Documentation**
   - Photos of archaeological findings from excavations

### Database Connection
- **Backend**: MySQL 10.4 (MariaDB)
- **Frontend**: Budibase (low-code dashboard builder)
- **Connection**: Direct MySQL connection with read-only credentials for dashboard

*Note: Database server is no longer active (professor's server shut down), but schema and queries remain available for reproducibility.*

---

## Setup (Local)

### Prerequisites
- MySQL 8.0+ or MariaDB 10.4+
- Budibase (Docker or cloud instance)
- phpMyAdmin (optional, for GUI)

### Installation

#### 1. Setup Database
```bash
# Login to MySQL
mysql -u root -p

# Import database
mysql -u root -p < arkeologidb.sql

# Verify import
mysql -u root -p arkeologidb
SHOW TABLES;
```

#### 2. Test Queries
```bash
# Run sample queries
mysql -u root -p arkeologidb < queries.sql
```

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| **Database** | MySQL 10.4 (MariaDB) |
| **Normalization** | Third Normal Form (3NF) |
| **Dashboard** | Budibase (Low-code platform) |
| **Query Language** | SQL (SELECT, JOIN, GROUP BY, aggregations) |
| **Documentation** | ERD (Crow's foot notation), Data Dictionary |
| **Version Control** | Git, GitHub |

---

## Sample Data

Database populated with simulation data:
- **5 Archaeologists** from various institutions (UNNES, UI, UGM)
- **9 Archaeological Sites** in Indonesia (Borobudur, Gunung Padang, Trowulan, etc)
- **11 Artifacts** with cultural period classification
- **8 Research Expeditions** (July-Sept 2023, June 2025)
- **6 Lab Results** (morphology, composition, dating analysis)

---

## Limitations & Future Work

### Limitations
1. **No authentication/authorization** — Database lacks user management for access control
2. **Static dashboard** — Budibase dashboard doesn't auto-refresh (needs manual reload)
3. **Limited sample data** — Only 11 artifacts for demonstration; real-world would have thousands
4. **No GIS integration** — Geographic location still text field, no map visualization

### Future Work
1. **Add user roles** — Implement RBAC (archaeologist, lab analyst, public viewer)
2. **GIS integration** — Integrate Google Maps/Leaflet for site visualization
3. **Image storage** — Add artifact_photos table with blob storage or S3
4. **Time-series analysis** — Track artifact condition changes over time (conservation monitoring)
5. **Export reports** — Auto-generate PDF expedition reports from database

---

## License

- **Database Schema**: Open for educational purposes
- **Sample Data**: Simulation for academic project
- **Budibase**: Open-source (GPL v3)
