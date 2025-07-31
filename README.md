# Personal Finance Webapp
A full-stack personal finance management tool that helps users understand and improve their financial health through rich data visualisation, automated insights, and powerful transaction handling — all in a secure environment.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Code Snippets](#code-snippets)
- [Screenshots](#screenshots)
- [Purpose](#purpose)
- [Security Note](#security-note)

---

## Features

### User Authentication
- Secure sign-up and login
- All passwords hashed with industry standards

### Dashboard with Real-Time Insights
- Net balance overview
- Upcoming bills
- Visual graphs for spending and income patterns
- Auto-generated financial insights

### Transactions
- Sortable, paginated transaction list
- Excel file upload for batch imports
- Auto-categorisation with a custom ML model

### Accounts Management
- Normal and savings accounts
- Savings include 15-year compound interest projections

### Budgets & Bills
- Set one budget per category
- Track and visualise recurring bills

### Profile
- View and manage user profile and personal details

---

## Tech Stack

- **Frontend**: React.js, TailwindCSS, Chart.js
- **Backend**: Django REST Framework
- **Database**: PostgreSQL
- **Authentication**: JWT (JSON Web Tokens)
- **ML**: Scikit-learn for transaction categorisation
- **File Upload**: Pandas for Excel processing

---

## Code Snippets
### Adding transactions:
This is the API endpoint for adding new transactions to the database

```python
@api_bp.route("/transactions/add/single", methods=["POST"])
@login_required
def transaction_add():
    new_transaction = request.form.to_dict()
    try:
        result = add_transaction(new_transaction)
        return jsonify({"status": "Transaction has been added successfully.", "inserted": result}), 200
    except ValueError as ve:
        logging.error("An error occured:\n%s", traceback.format_exc())
        return jsonify({"error": str(ve)}), 400
    except Exception as e:
        logging.error("An unexpected error occured:\n%s", traceback.format_exc())
        return jsonify({"error": "Unexpected server error."}), 500
```

This helper function handles the core transaction adding logic. It converts input types, predicts the category using an ML model, updates account balances and budgets, and safely commits the transaction to the database.

```python
def add_transaction(new_transaction: Dict[str, Any]) -> Dict[str, Any]:
    #Checking required fields
    check_transaction_required_fields(new_transaction)
    #Handling type conversions
    new_transaction["amount"] = Decimal(new_transaction["amount"])
    new_transaction["date"] = datetime.strptime(new_transaction["date"], "%d/%m/%Y %H:%M")

    #Getting correct account id from account name and updating account table
    account: int = get_or_create_account(new_transaction["account_name"])
    new_transaction["account_id"] = account._id

    #Predicting category
    category_name: str = predict_category(new_transaction["description"], new_transaction["transaction_type"], new_transaction["amount"])

    #Getting correct category id from category name
    new_transaction["category_id"] = get_category(category_name)
    
    #Updating account amount
    update_account_balance(account._id, new_transaction["amount"], new_transaction["transaction_type"])

    #Update any relevent budget
    if new_transaction["transaction_type"] == "Expense":
        update_budget(new_transaction["amount"], category_name)

    #Add current user
    new_transaction["user_id"] = int(current_user.get_id())
        
    #Sanitising and creating a model instance by unpacking
    new_transaction.pop("account_name", None)
    sanitised_transaction: Dict[str, Any] = sanitise_model_dict(new_transaction, Transaction)
    unpacked_transaction = Transaction(**sanitised_transaction)

    try:
        db.session.add(unpacked_transaction)
        db.session.commit()
        return sanitised_transaction
    except Exception as e:
        db.session.rollback()
        raise ValueError("Failed to add transaction to database.") from e
```

---

## Screenshots

---

## Purpose

---

## Security Note
