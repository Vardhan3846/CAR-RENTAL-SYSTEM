#include <iostream>
#include <mysql/mysql.h>

using namespace std;

const char* HOST = "localhost";
const char* USER = "root";
const char* PASSWORD = "password";
const char* DATABASE = "car_rental";

MYSQL* connectDB() {
    MYSQL* conn = mysql_init(NULL);
    if (conn == NULL) {
        cerr << "MySQL Initialization Failed" << endl;
        exit(1);
    }
    
    conn = mysql_real_connect(conn, HOST, USER, PASSWORD, DATABASE, 0, NULL, 0);
    if (conn == NULL) {
        cerr << "MySQL Connection Failed: " << mysql_error(conn) << endl;
        exit(1);
    }
    return conn;
}

void addCar(MYSQL* conn) {
    string model;
    double price;
    cout << "Enter Car Model: ";
    cin >> model;
    cout << "Enter Rent Price per Day: ";
    cin >> price;
    
    string query = "INSERT INTO cars (model, price_per_day, available) VALUES ('" + model + "', " + to_string(price) + ", 1)";
    if (mysql_query(conn, query.c_str()) == 0)
        cout << "Car Added Successfully!" << endl;
    else
        cerr << "Failed to Add Car: " << mysql_error(conn) << endl;
}

void rentCar(MYSQL* conn) {
    int id;
    cout << "Enter Car ID to Rent: ";
    cin >> id;
    
    string query = "UPDATE cars SET available = 0 WHERE id = " + to_string(id) + " AND available = 1";
    if (mysql_query(conn, query.c_str()) == 0 && mysql_affected_rows(conn) > 0)
        cout << "Car Rented Successfully!" << endl;
    else
        cerr << "Car Not Available or Invalid ID." << endl;
}

void returnCar(MYSQL* conn) {
    int id;
    cout << "Enter Car ID to Return: ";
    cin >> id;
    
    string query = "UPDATE cars SET available = 1 WHERE id = " + to_string(id);
    if (mysql_query(conn, query.c_str()) == 0 && mysql_affected_rows(conn) > 0)
        cout << "Car Returned Successfully!" << endl;
    else
        cerr << "Invalid Car ID." << endl;
}

void viewAvailableCars(MYSQL* conn) {
    string query = "SELECT * FROM cars WHERE available = 1";
    if (mysql_query(conn, query.c_str()) == 0) {
        MYSQL_RES* res = mysql_store_result(conn);
        MYSQL_ROW row;
        
        cout << "Available Cars: " << endl;
        while ((row = mysql_fetch_row(res))) {
            cout << "ID: " << row[0] << " | Model: " << row[1] << " | Price Per Day: " << row[2] << endl;
        }
        mysql_free_result(res);
    } else {
        cerr << "Failed to Fetch Cars: " << mysql_error(conn) << endl;
    }
}

int main() {
    MYSQL* conn = connectDB();
    int choice;
    
    while (true) {
        cout << "\nCar Rental System";
        cout << "\n1. Add Car";
        cout << "\n2. Rent Car";
        cout << "\n3. Return Car";
        cout << "\n4. View Available Cars";
        cout << "\n5. Exit";
        cout << "\nEnter Choice: ";
        cin >> choice;
        
        switch (choice) {
            case 1: addCar(conn); break;
            case 2: rentCar(conn); break;
            case 3: returnCar(conn); break;
            case 4: viewAvailableCars(conn); break;
            case 5: mysql_close(conn); return 0;
            default: cout << "Invalid Choice!" << endl;
        }
    }
}
