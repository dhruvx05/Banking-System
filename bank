import random
import sys
import sqlite3

# Initialize random seed
random.seed()

# Setting up the database connection and cursor
conn = sqlite3.connect('card.s3db')
cur = conn.cursor()
cur.execute("""
CREATE TABLE IF NOT EXISTS card (
    id INTEGER PRIMARY KEY,
    number TEXT UNIQUE,
    pin TEXT,
    balance INTEGER DEFAULT 0
);""")
conn.commit()

class Card:
    def __init__(self):
        self.card = ''
        self.pin = ''
        self.login_card = ''
        self.login_pin = ''
        self.balance = 0

    def create_account(self):
        """Creates a new account with a card number and PIN."""
        self.card = '400000' + str(random.randint(100000000, 999999999))
        self.card = self.luhn()
        self.pin = f"{random.randint(1000, 9999)}"
        print("\nYour card has been created")
        print(f"Your card number:\n{self.card}")
        print(f"Your card PIN:\n{self.pin}")
        
        try:
            cur.execute("INSERT INTO card (number, pin) VALUES (?, ?);", (self.card, self.pin))
            conn.commit()
        except sqlite3.IntegrityError:
            print("Error: Failed to create account. Please try again.")

    def log_in(self):
        """Logs into an existing account."""
        self.login_card = input("\nEnter your card number:\n")
        self.login_pin = input("Enter your PIN:\n")
        cur.execute("SELECT balance FROM card WHERE number = ? AND pin = ?", (self.login_card, self.login_pin))
        row = cur.fetchone()
        if row:
            self.balance = row[0]
            print("\nYou have successfully logged in")
            self.success()
        else:
            print("\nWrong card number or PIN")

    def success(self):
        """Handles logged-in user operations."""
        while True:
            print("""\n1. Balance
2. Add income
3. Do transfer
4. Close account
5. Log out
0. Exit""")
            choice = input()
            if choice == '1':
                print(f"\nBalance: {self.balance}")
            elif choice == '2':
                self.add_income()
            elif choice == '3':
                self.do_transfer()
            elif choice == '4':
                self.close_account()
                break
            elif choice == '5':
                print("\nYou have successfully logged out!")
                break
            elif choice == '0':
                print("\nBye!")
                conn.close()
                sys.exit()
            else:
                print("Invalid choice. Please try again.")

    def add_income(self):
        """Adds income to the account."""
        try:
            amount = int(input("\nEnter income:\n"))
            self.balance += amount
            cur.execute("UPDATE card SET balance = ? WHERE number = ?", (self.balance, self.login_card))
            conn.commit()
            print("Income was added!")
        except ValueError:
            print("Invalid input. Please enter a valid number.")

    def do_transfer(self):
        """Performs a money transfer to another card."""
        receiver_card = input("\nEnter card number:\n")
        if not self.luhn_validate(receiver_card):
            print("Probably you made a mistake in the card number. Please try again!")
            return
        cur.execute("SELECT balance FROM card WHERE number = ?", (receiver_card,))
        receiver = cur.fetchone()
        if not receiver:
            print("Such a card does not exist.")
            return
        try:
            transfer_amount = int(input("Enter how much money you want to transfer:\n"))
            if transfer_amount > self.balance:
                print("Not enough money!")
            else:
                self.balance -= transfer_amount
                receiver_balance = receiver[0] + transfer_amount
                cur.execute("UPDATE card SET balance = ? WHERE number = ?", (self.balance, self.login_card))
                cur.execute("UPDATE card SET balance = ? WHERE number = ?", (receiver_balance, receiver_card))
                conn.commit()
                print("Success!")
        except ValueError:
            print("Invalid input. Please enter a valid amount.")

    def close_account(self):
        """Closes the logged-in user's account."""
        cur.execute("DELETE FROM card WHERE number = ?", (self.login_card,))
        conn.commit()
        print("\nThe account has been closed!")

    def luhn_validate(self, num):
        """Validates a card number using the Luhn algorithm."""
        digits = [int(x) for x in num]
        for i in range(0, len(digits) - 1, 2):
            digits[i] *= 2
            if digits[i] > 9:
                digits[i] -= 9
        return sum(digits) % 10 == 0

    def luhn(self):
        """Generates a valid card number using the Luhn algorithm."""
        digits = [int(x) for x in self.card]
        for i in range(0, len(digits), 2):
            digits[i] *= 2
            if digits[i] > 9:
                digits[i] -= 9
        checksum = (10 - sum(digits) % 10) % 10
        return self.card + str(checksum)

    def menu(self):
        """Displays the main menu."""
        while True:
            print("""\n1. Create an account
2. Log into account
0. Exit""")
            choice = input()
            if choice == '1':
                self.create_account()
            elif choice == '2':
                self.log_in()
            elif choice == '0':
                conn.close()
                print("\nBye!")
                break
            else:
                print("Invalid choice. Please try again.")

# Run the application
card = Card()
card.menu()
