#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <iomanip>
#include <ctime>
#include <sstream>
using namespace std;

struct Transaction {
    string type;
    double amount;
    string timestamp;
};

struct Account {
    int accountNumber;
    string name;
    string phone;
    double balance;
    vector<Transaction> transactions;
};

const string FILENAME = "accounts.dat";
const string ADMIN_USER = "admin";
const string ADMIN_PASS = "1234";

string currentTimestamp() {
    time_t now = time(nullptr);
    tm *ltm = localtime(&now);
    char buf[30];
    strftime(buf, sizeof(buf), "%Y-%m-%d %H:%M:%S", ltm);
    return string(buf);
}

bool adminLogin() {
    string user, pass;
    cout << "\n";
    cout << "Username: "; cin >> user;
    cout << "Password: "; cin >> pass;
    return user == ADMIN_USER && pass == ADMIN_PASS;
}

vector<Account> loadAccounts() {
    vector<Account> accounts;
    ifstream file(FILENAME, ios::binary);
    if (file) {
        while (file.peek() != EOF) {
            Account acc;
            size_t transCount;
            file.read((char*)&acc, sizeof(Account) - sizeof(vector<Transaction>));
            file.read((char*)&transCount, sizeof(size_t));
            acc.transactions.resize(transCount);
            for (size_t i = 0; i < transCount; ++i) {
                size_t len;
                file.read((char*)&len, sizeof(size_t));
                acc.transactions[i].type.resize(len);
                file.read(&acc.transactions[i].type[0], len);
                file.read((char*)&acc.transactions[i].amount, sizeof(double));
                file.read((char*)&len, sizeof(size_t));
                acc.transactions[i].timestamp.resize(len);
                file.read(&acc.transactions[i].timestamp[0], len);
            }
            accounts.push_back(acc);
        }
        file.close();
    }
    return accounts;
}

void saveAccounts(const vector<Account>& accounts) {
    ofstream file(FILENAME, ios::binary | ios::trunc);
    for (const auto& acc : accounts) {
        file.write((char*)&acc, sizeof(Account) - sizeof(vector<Transaction>));
        size_t transCount = acc.transactions.size();
        file.write((char*)&transCount, sizeof(size_t));
        for (const auto& t : acc.transactions) {
            size_t len = t.type.size();
            file.write((char*)&len, sizeof(size_t));
            file.write(t.type.c_str(), len);
            file.write((char*)&t.amount, sizeof(double));
            len = t.timestamp.size();
            file.write((char*)&len, sizeof(size_t));
            file.write(t.timestamp.c_str(), len);
        }
    }
    file.close();
}

void logTransaction(Account &acc, const string &type, double amount) {
    acc.transactions.push_back({type, amount, currentTimestamp()});
}

void createAccount() {
    Account acc;
    cout << "\n";
    cout << "Enter Account Number: "; cin >> acc.accountNumber;
    cout << "Enter Name: "; cin.ignore(); getline(cin, acc.name);
    cout << "Enter Phone: "; getline(cin, acc.phone);
    cout << "Enter Initial Balance: "; cin >> acc.balance;

    vector<Account> accounts = loadAccounts();
    for (const auto& a : accounts) {
        if (a.accountNumber == acc.accountNumber) {
            cout << "Account already exists!\n";
            return;
        }
    }
    logTransaction(acc, "Account Created", acc.balance);
    accounts.push_back(acc);
    saveAccounts(accounts);
    cout << "Account created successfully!\n";
}

int findAccountIndex(int accNo, vector<Account>& accounts) {
    for (int i = 0; i < accounts.size(); ++i)
        if (accounts[i].accountNumber == accNo) return i;
    return -1;
}

void modifyAccount() {
    int accNo;
    cout << "\n";
    cout << "Enter Account Number: "; cin >> accNo;
    vector<Account> accounts = loadAccounts();
    int idx = findAccountIndex(accNo, accounts);
    if (idx == -1) {
        cout << "Account not found!\n";
        return;
    }
    cout << "Enter New Name: "; cin.ignore(); getline(cin, accounts[idx].name);
    cout << "Enter New Phone: "; getline(cin, accounts[idx].phone);
    saveAccounts(accounts);
    cout << "Account modified successfully!\n";
}

void depositWithdraw(bool isDeposit) {
    int accNo;
    double amount;
    cout << "\n";
    cout << "Enter Account Number: "; cin >> accNo;
    vector<Account> accounts = loadAccounts();
    int idx = findAccountIndex(accNo, accounts);
    if (idx == -1) {
        cout << "Account not found!\n";
        return;
    }
    cout << "Enter amount: "; cin >> amount;
    if (!isDeposit && accounts[idx].balance < amount) {
        cout << "Insufficient balance!\n";
        return;
    }
    accounts[idx].balance += (isDeposit ? amount : -amount);
    logTransaction(accounts[idx], isDeposit ? "Deposit" : "Withdraw", amount);
    saveAccounts(accounts);
    cout << "Transaction successful!\n";
}

void transferFunds() {
    int fromAcc, toAcc;
    double amount;
    cout << "\n";
    cout << "From Account Number: "; cin >> fromAcc;
    cout << "To Account Number: "; cin >> toAcc;
    cout << "Amount: "; cin >> amount;

    vector<Account> accounts = loadAccounts();
    int fromIdx = findAccountIndex(fromAcc, accounts);
    int toIdx = findAccountIndex(toAcc, accounts);
    if (fromIdx == -1 || toIdx == -1) {
        cout << "One or both accounts not found!\n";
        return;
    }
    if (accounts[fromIdx].balance < amount) {
        cout << "Insufficient balance!\n";
        return;
    }
    accounts[fromIdx].balance -= amount;
    accounts[toIdx].balance += amount;
    logTransaction(accounts[fromIdx], "Transfer Sent", amount);
    logTransaction(accounts[toIdx], "Transfer Received", amount);
    saveAccounts(accounts);
    cout << "Transfer successful!\n";
}

void searchAccount() {
    int accNo;
    cout << "\n";
    cout << "Enter Account Number: "; cin >> accNo;
    vector<Account> accounts = loadAccounts();
    int idx = findAccountIndex(accNo, accounts);
    if (idx == -1) {
        cout << "Account not found!\n";
        return;
    }
    Account& acc = accounts[idx];
    cout << "\n";
    cout << "Number: " << acc.accountNumber << "\nName: " << acc.name
         << "\nPhone: " << acc.phone << "\nBalance: ₹" << fixed << setprecision(2) << acc.balance << "\n";
}

void viewPassbook() {
    int accNo;
    cout << "\n";
    cout << "Enter Account Number: "; cin >> accNo;
    vector<Account> accounts = loadAccounts();
    int idx = findAccountIndex(accNo, accounts);
    if (idx == -1) {
        cout << "Account not found!\n";
        return;
    }
    const auto& txns = accounts[idx].transactions;
    cout << "\n";
    cout << left << setw(20) << "Type" << setw(15) << "Amount" << "Date & Time\n";
    cout << "-----------------------------------------------\n";
    for (const auto& txn : txns) {
        cout << left << setw(20) << txn.type << "₹" << setw(14) << txn.amount << txn.timestamp << "\n";
    }
}

void menu() {
    int choice;
    do {
        cout << "\n";
        cout << "1. Create Account\n2. Modify Account\n3. Deposit\n4. Withdraw\n";
        cout << "5. Transfer Funds\n6. Search Account\n7. View Passbook\n0. Exit\n";
        cout << "Enter choice: "; cin >> choice;
        switch (choice) {
            case 1: createAccount(); break;
            case 2: modifyAccount(); break;
            case 3: depositWithdraw(true); break;
            case 4: depositWithdraw(false); break;
            case 5: transferFunds(); break;
            case 6: searchAccount(); break;
            case 7: viewPassbook(); break;
            case 0: cout << "\n"; break;
            default: cout << "Invalid option!\n";
        }
    } while (choice != 0);
}

int main() {
    cout << "\n";
    if (!adminLogin()) {
        cout << "Invalid login. Exiting.\n";
        return 0;
    }
    menu();
    return 0;
}
