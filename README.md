
# PostgreSQL 16 with `pg_hint_plan` and Cardinality Injection Support

This project provides a streamlined setup for PostgreSQL 16 with [`pg_hint_plan`](https://github.com/ossc-db/pg_hint_plan), along with a loader to initialize a sample database using the IMDb dataset.

In addition to supporting planner hints, we also allow **injecting custom cardinality estimates**. This feature enables users to manually specify estimated row counts for intermediate query results, helping PostgreSQL generate more accurate and efficient execution plans — especially in cases where default estimates fall short.

To support benchmarking and experimentation, we provide a configurable switch:
users can choose whether to supply their own cardinalities or rely entirely on PostgreSQL’s default estimates. This makes it easy to **compare performance** and understand how better cardinality can influence plan selection and overall query efficiency.

---

## 1. Clone and Install PostgreSQL 16

```bash
# In your working directory
git clone https://github.com/Hap-Hugh/PG16.git
cd PG16

# Configure with optional installation path
./configure --prefix=/usr/local/pgsql/16.2

# Add --without-icu if ICU library problems occur
./configure --prefix=/usr/local/pgsql/16.2 --without-icu

# Build and install
make -j
make install
````

### Set environment variables

```bash
export PGHOME=/usr/local/pgsql/16.2   # use your actual prefix path
export PATH="$PGHOME/bin:$PATH"
```

---

## 2. Install `pg_hint_plan`

```bash
# In your working directory
git clone https://github.com/ossc-db/pg_hint_plan.git -b REL16_1_6_0
cd pg_hint_plan

make -j
make install
```

---

## 3. Initialize and Start PostgreSQL Database

```bash
# Clean up previous instance if exists
rm -rf ./imdb

# Initialize database
pg_ctl -D ./imdb initdb

# Use the provided PostgreSQL config
cp ./PG16/postgresql.conf ./imdb

# Start the database
pg_ctl -D ./imdb -l ./dblogs/logfile start
```

---

## 4. Download and Extract IMDb Dataset

```bash
# Download IMDb dataset
wget https://event.cwi.nl/da/job/imdb.tgz

# Extract into a clean datasource directory
mkdir ./datasource
cd ./datasource
tar zxvf ../imdb.tgz
```

---

## 5. Load IMDb Data using Loader (from [Balsa](https://github.com/balsa-project/balsa))

```bash
# In your working directory
cd ./PG16

# Run the Python script to prepend IMDb header
python3 ./prepend_imdb_headers.py

# Run the loader script
./load_job_postgres.sh [full path to your datasource directory]
```

---

## Notes

* Make sure the port (`PGPORT`) matches your PostgreSQL instance.
* The default DB name created is `imdbload`.
* Please copy the PG16 config file `./PG16/postgresql.conf` to the `imdb` directory.

## Reference 

We welcome all forms of discussion and collaboration for future research and are eager to help with plan selection in your project. If you find this repository valuable, please consider citing our work:
~~~
@article{xiu2024parqo,
      title={{PARQO}: Penalty-Aware Robust Plan Selection in Query Optimization}, 
      author={Haibo Xiu and Pankaj K. Agarwal and Jun Yang},
      year={2024},
      journal={Proceedings of the VLDB Endowment},
      volume={17},
      number={13},
}
~~~
