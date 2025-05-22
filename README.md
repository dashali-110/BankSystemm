# BankSystemm

import java.io.*;
import java.util.*;

class BankAccount implements Serializable {
    private static final long serialVersionUID = 1L;
    private String accountNumber;
    private double balance;

    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    public String getAccountNumber() {
        return accountNumber;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }

    @Override
    public String toString() {
        return "Account Number: " + accountNumber + ", Balance: " + balance;
    }
}

class BankSystem {
    private Map<String, BankAccount> accounts = new HashMap<>();
    private Map<String, Double> exchangeRates = new HashMap<>();
    private final String ACCOUNTS_FILE = "accounts.txt";
    private final String EXCHANGE_FILE = "exchange_rates.txt";

    public BankSystem() {
        loadAccountsFromFile();
        loadExchangeRates();
    }

    public void createAccount(String accountNumber, double initialBalance) {
        if (!accounts.containsKey(accountNumber)) {
            BankAccount account = new BankAccount(accountNumber, initialBalance);
            accounts.put(accountNumber, account);
            saveAccountsToFile();
            System.out.println("Bank account created: " + account);
        } else {
            System.out.println("This account number already exists!");
        }
    }

    public void deposit(String accountNumber, double amount) {
        BankAccount account = accounts.get(accountNumber);
        if (account != null) {
            account.deposit(amount);
            saveAccountsToFile();
            System.out.println("Deposit completed. New balance: " + account.getBalance());
        } else {
            System.out.println("Account not found!");
        }
    }

    public void withdraw(String accountNumber, double amount) {
        BankAccount account = accounts.get(accountNumber);
        if (account != null && account.withdraw(amount)) {
            saveAccountsToFile();
            System.out.println("Successful withdrawal! New balance: " + account.getBalance());
        } else {
            System.out.println("Withdrawal failed! Insufficient balance or account not found.");
        }
    }

    public void checkBalance(String accountNumber) {
        BankAccount account = accounts.get(accountNumber);
        if (account != null) {
            System.out.println("Account balance: " + account.getBalance());
        } else {
            System.out.println("Account not found!");
        }
    }

    private void saveAccountsToFile() {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(ACCOUNTS_FILE))) {
            oos.writeObject(accounts);
        } catch (IOException e) {
            System.out.println("Error saving accounts file!");
        }
    }

    private void loadAccountsFromFile() {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(ACCOUNTS_FILE))) {
            accounts = (HashMap<String, BankAccount>) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            System.out.println("Accounts file not found. Creating new file...");
        }
    }

    private void loadExchangeRates() {
        try (BufferedReader reader = new BufferedReader(new FileReader(EXCHANGE_FILE))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] parts = line.split(",");
                if (parts.length == 2) {
                    exchangeRates.put(parts[0], Double.parseDouble(parts[1]));
                }
            }
        } catch (IOException e) {
            System.out.println("Exchange rates file not found!");
        }
    }

    public void convertCurrency(String date, double amount, boolean isRialToDollar) {
        if (!exchangeRates.containsKey(date)) {
            System.out.println("Exchange rate for this date not found!");
            return;
        }

        double rate = exchangeRates.get(date);
        if (isRialToDollar) {
            double converted = amount / rate;
            System.out.println(amount + " Rial equals " + converted + " USD on " + date);
        } else {
            double converted = amount * rate;
            System.out.println(amount + " USD equals " + converted + " Rial on " + date);
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankSystem bank = new BankSystem();
        Scanner scanner = new Scanner(System.in);

        bank.createAccount("12345", 1000);
        bank.deposit("12345", 500);
        bank.withdraw("12345", 300);
        bank.checkBalance("12345");

        System.out.println("\nEnter the desired date (e.g., 2025-05-22): ");
        String selectedDate = scanner.nextLine();

        System.out.println("Enter the amount: ");
        double amount = scanner.nextDouble();

        System.out.println("Choose: 1) Rial to Dollar | 2) Dollar to Rial");
        int choice = scanner.nextInt();

        bank.convertCurrency(selectedDate, amount, choice == 1);

        scanner.close();
    }
}
