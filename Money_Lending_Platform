from datetime import datetime, timedelta
import gspread
import pandas as pd
from google.colab import auth
from google.auth import default
from datetime import datetime
pd.set_option('display.max_columns', None)
pd.set_option('display.width', 1000)



auth.authenticate_user()
creds, _ = default()
gs = gspread.authorize(creds)
worksheet = gs.open('Debtors').sheet1

def add_debtor():
    action = input("Do you want to add a new debtor (A) or remove an existing one (R)? ").strip().lower()

    if action == 'a':
        name = input("Enter debtor's name: ")
        date = input("Enter date when money was taken (MM/DD/YYYY): ")
        amount = input("Enter amount owed: ")
        interest = input("Enter interest rate: ")
        phone_number = input("Enter phone number: ")
        address = input("Enter address: ")
        choice = input("Enter option number (1-3) for payment status:\n1. Pending\n2. Paid\n3. Overdue\nChoice: ")
        payment_status = ""
        if choice == '1':
            payment_status = "Pending"
        elif choice == '2':
            payment_status = "Paid"
        elif choice == '3':
            payment_status = "Overdue"
        else:
            print("Invalid option. Setting payment status to Pending.")
            payment_status = "Pending"
        due_date = input("Enter payment due date (MM/DD/YYYY): ")
        id_number = input("Enter ID number: ")

        worksheet.append_row([name, date, amount, interest, phone_number, address, payment_status, due_date, id_number])
        print("Debtor added successfully!")
    elif action == 'r':
        debtor_name = input("Enter debtor's name to remove: ").strip()
        phone_number = input("Enter debtor's phone number (optional, press Enter to skip): ").strip()

        data = worksheet.get_all_values()
        headers = data[0] if data else []

        matching_debtors = []

        for row in data[1:]:
            if row[0] == debtor_name:
                if phone_number == '' or row[4] == phone_number:
                    matching_debtors.append(row)

        if not matching_debtors:
            print("No matching debtors found.")
            return

        print("Matching debtors found:")
        df = pd.DataFrame(matching_debtors, columns=headers)
        print(df)

        choice = input("Enter the index of the debtor to remove (enter 'cancel' to cancel): ").strip()
        if choice.isdigit() and 1 <= int(choice) <= len(matching_debtors):
            index_to_remove = int(choice) - 1
            debtor_to_remove = matching_debtors[index_to_remove]


            worksheet.delete_row(index_to_remove + 2)
            print(f"Debtor '{debtor_to_remove[0]}' removed successfully!")
        else:
            print("Invalid index. No changes made.")
    else:
        print("Invalid action. Please choose 'A' to add a new debtor or 'R' to remove an existing one.")


def modify_debtor():
    print("Enter criteria to identify the debtor to modify:")
    debtor_name = input("Debtor's name: ")
    phone_number = input("Debtor's phone number (optional, press Enter to skip): ")

    data = worksheet.get_all_values()
    headers = data[0] if data else []

    matching_debtors = []

    for row in data[1:]:
        if row[0] == debtor_name:
            if phone_number == '' or row[4] == phone_number:
                matching_debtors.append(row)

    if not matching_debtors:
        print("No matching debtors found.")
        return

    print("Matching debtors found:")
    df = pd.DataFrame(matching_debtors, columns=headers)
    print(df)

    try:
        choice = int(input("Select the debtor to modify (enter option number): "))
        if 1 <= choice <= len(matching_debtors):
            selected_debtor = matching_debtors[choice - 1]

            print(f"Select column to modify for '{selected_debtor[0]}':")
            for idx, header in enumerate(headers):
                print(f"{idx + 1}. {header}")

            try:
                col_choice = int(input(f"Enter option number (1-{len(headers)}): "))
                if 1 <= col_choice <= len(headers):
                    column_name = headers[col_choice - 1]
                    new_value = input(f"Enter new value for '{column_name}': ")


                    for row in data[1:]:
                        if row == selected_debtor:
                            row[col_choice - 1] = new_value


                    worksheet.clear()


                    for row in data:
                        worksheet.append_row(row)

                    print("Debtor information updated successfully!")
                else:
                    print("Invalid column option.")
            except ValueError:
                print("Invalid column option. Please enter a valid number.")
        else:
            print("Invalid debtor option.")
    except ValueError:
        print("Invalid debtor option. Please enter a valid number.")

def calculate_total_amount_with_interest(data):
    total_amount_with_interest = 0

    for row in data[1:]:
        amount = float(row[2])
        interest_rate = float(row[3])

        amount_with_interest = amount * (1 + (interest_rate / 100))
        total_amount_with_interest += amount_with_interest

    return total_amount_with_interest

def view_debtors():
    data = worksheet.get_all_values()

    if len(data) > 1:
        headers = data[0]
        df = pd.DataFrame(data[1:], columns=headers)


        df['Amount'] = df['Amount'].astype(float)

        df['Interest'] = df['Interest'].astype(float)

        df['Amount_to_be_Paid'] = df['Amount'] * (1 + df['Interest'] / 100)


        print(df)


        updated_data = [df.columns.tolist()] + df.values.tolist()


        worksheet.clear()


        for row in updated_data:
            worksheet.append_row(row)

        print("\nAmount_to_be_Paid updated successfully in the worksheet.")
    else:
        print("No debtors found.")




def update_interest_if_overdue():
    today = datetime.now().date()
    data = worksheet.get_all_values()

    for row in data[1:]:
        due_date_str = row[7]
        if due_date_str:
            due_date = datetime.strptime(due_date_str, "%m/%d/%Y").date()
            if due_date < today:

                new_due_date = today + timedelta(days=60)


                row[7] = new_due_date.strftime("%m/%d/%Y")


                current_interest = float(row[3])
                row[3] = str(current_interest * 2)


                row[6] = "Overdue"

    worksheet.clear()

    for row in data:
        worksheet.append_row(row)

    print("Interest updated for overdue debtors. Due dates adjusted to two months from current date.")

def main():
    print(" *** Welcome to Debt Management System CLI *** ")
    print("Commands: add, view, modify, update interest, exit\n\n")

    while True:
        command = input("\nEnter command: ").strip().lower()

        if command == 'add':
            add_debtor()
        elif command == 'view':
            view_debtors()
        elif command == 'modify':
            modify_debtor()
        elif command == 'update interest':
            update_interest_if_overdue()
        elif command == 'exit':
            print("Exiting program.")
            break
        else:
            print("Invalid command. Please try again.")

if __name__ == "__main__":
    main()
