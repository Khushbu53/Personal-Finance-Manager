🧱 Project Structure
 finance_manager/
│
├── main.py # Entry point
├── expense.py # Expense class
├── file_manager.py # File handling & backup
├── menu.py # CLI menu system
├── reports.py # Reports & analytics
├── utils.py # Validation utilities
│
├── data/
│ └── expenses.csv # Expense storage
│
├── backups/ # Backup files
└── reports/ # Generated reports

# Personal-Finance-Manager
import json
import os
from datetime import datetime
from collections import defaultdict

class FinanceManager:
    def __init__(self, filename='expenses.json'):
        self.filename = filename
        self.data = self.load_data()

    def load_data(self):
        if os.path.exists(self.filename):
            with open(self.filename, 'r') as f:
                return json.load(f)
        return []

    def save_data(self):
        with open(self.filename, 'w') as f:
            json.dump(self.data, f, indent=4)

    def add_expense(self):
        print("\n--- Add New Expense ---")
        try:
            amount = float(input("Enter amount: "))
            category = input("Enter category (e.g., Food, Transport, Bills): ").strip().title()
            desc = input("Enter description: ").strip()
            date = input("Enter date (YYYY-MM-DD) or press Enter for today: ").strip()
            
            if not date:
                date = datetime.now().strftime("%Y-%m-%d")
            
            # Validate date format
            datetime.strptime(date, "%Y-%m-%d")
            
            entry = {"date": date, "amount": amount, "category": category, "description": desc}
            self.data.append(entry)
            self.save_data()
            print("✅ Expense added successfully!")
        except ValueError:
            print("❌ Invalid input. Amount must be a number and date must be YYYY-MM-DD.")

    def view_all(self):
        print(f"\n{'Date':<12} | {'Category':<15} | {'Amount':<10} | {'Description'}")
        print("-" * 60)
        for item in sorted(self.data, key=lambda x: x['date']):
            print(f"{item['date']:<12} | {item['category']:<15} | ${item['amount']:<9.2f} | {item['description']}")

    def category_summary(self):
        summary = defaultdict(float)
        for item in self.data:
            summary[item['category']] += item['amount']
        
        print("\n--- Category-wise Summary ---")
        for cat, total in summary.items():
            print(f"{cat:<15}: ${total:,.2f}")

    def monthly_report(self):
        month_input = input("\nEnter month (YYYY-MM): ").strip()
        report = [item for item in self.data if item['date'].startswith(month_input)]
        
        if not report:
            print("No data found for that month.")
            return
        
        total = sum(item['amount'] for item in report)
        print(f"\n--- Report for {month_input} ---")
        print(f"Total Transactions: {len(report)}")
        print(f"Total Spent: ${total:,.2f}")

    def search_expenses(self):
        query = input("\nEnter search term (Category or Description): ").lower()
        results = [i for i in self.data if query in i['category'].lower() or query in i['description'].lower()]
        
        if results:
            for r in results:
                print(f"{r['date']} | {r['category']} | ${r['amount']} | {r['description']}")
        else:
            print("No matching expenses found.")

    def backup_data(self):
        backup_name = f"backup_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
        with open(backup_name, 'w') as f:
            json.dump(self.data, f, indent=4)
        print(f"✅ Backup created as {backup_name}")

def main():
    fm = FinanceManager()
    while True:
        print("\n--- MAIN MENU ---")
        print("1. Add New Expense")
        print("2. View All Expenses")
        print("3. View Category-wise Summary")
        print("4. Generate Monthly Report")
        print("5. Search Expenses")
        print("6. Backup Data")
        print("7. Exit")
        
        choice = input("Select an option (1-7): ")
        if choice == '1': fm.add_expense()
        elif choice == '2': fm.view_all()
        elif choice == '3': fm.category_summary()
        elif choice == '4': fm.monthly_report()
        elif choice == '5': fm.search_expenses()
        elif choice == '6': fm.backup_data()
        elif choice == '7': break
        else: print("Invalid choice.")

if __name__ == "__main__":
    main()
