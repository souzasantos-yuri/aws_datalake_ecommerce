Customers
├── customer_id NOT NULL
└── customer_id UNIQUE

Transactions
├── transaction_id NOT NULL
├── customer_id NOT NULL
└── amount >= 0

Transaction Products
├── transaction_id NOT NULL
├── product_id NOT NULL
└── quantity > 0

DQ Failure
    ↓
Glue Job FAILED
    ↓
Airflow Task FAILED
    ↓
Downstream Tasks Blocked