# My-best
# Supermarket Sale (simple POS)
11
A minimal Python supermarket sale / inventory + checkout demo using Flask + SQLite.

## Run
1. Create virtualenv: `python -m venv venv && source venv/bin/activate`
2. Install: `pip install -r requirements.txt`
3. Run: `python app.py`
4. Open http://127.0.0.1:5000

## Checkout
POST JSON list to `/checkout`:
# cli_pos.py
import json
import sqlite3
from pathlib import Path
DB = Path(__file__).parent / "supermarket.db"

def list_products(conn):
    cur = conn.execute("SELECT id, sku, name, price, qty FROM products")
    for r in cur.fetchall():
        print(r)

def checkout(conn, items):
    """
    items: list of (product_id, qty)
    """
    total = 0.0
    cur = conn.cursor()
    for pid, qty in items:
        cur.execute("SELECT id, name, price, qty FROM products WHERE id = ?", (pid,))
        row = cur.fetchone()
        if not row:
            raise ValueError(f"Product {pid} not found")
        if row[3] < qty:
            raise ValueError(f"Not enough stock for {row[1]}")
        line_total = row[2] * qty
        total += line_total
        cur.execute("UPDATE products SET qty = qty - ? WHERE id = ?", (qty, pid))
        print(f"{row[1]} x{qty} = {line_total:.2f}")
    conn.commit()
    print("TOTAL:", round(total,2))

if __name__ == "__main__":
    conn = sqlite3.connect(DB)
    print("Products:")
    list_products(conn)
    # sample checkout
    try:
        checkout(conn, [(1,2), (2,1)])  # buy 2 of id1, 1 of id2
    except Exception as e:
        print("Error:", e)
