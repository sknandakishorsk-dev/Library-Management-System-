# Library-Management-System-
#include <iostream>
#include <vector>
#include <memory>
#include <fstream>
#include <string>
#include <iomanip>

using namespace std;

// ======================================================
// BASE CLASS: MediaItem
// ======================================================

class MediaItem {
protected:
    int id;
    string title;
    bool checkedOut;

public:
    MediaItem(int id, string title)
        : id(id), title(title), checkedOut(false) {}

    virtual ~MediaItem() {}

    int getId() const {
        return id;
    }

    string getTitle() const {
        return title;
    }

    bool isCheckedOut() const {
        return checkedOut;
    }

    void checkout() {
        if (!checkedOut) {
            checkedOut = true;
            cout << "Item checked out successfully.\n";
        } else {
            cout << "Item is already checked out.\n";
        }
    }

    virtual void returnItem(int overdueDays) = 0;

    virtual void display() const = 0;

    // Used for saving data to file
    virtual string getType() const = 0;

    virtual void save(ofstream& file) const = 0;
};


// ======================================================
// DERIVED CLASS: Book
// ======================================================

class Book : public MediaItem {
private:
    string author;

public:
    Book(int id, string title, string author)
        : MediaItem(id, title), author(author) {}

    void returnItem(int overdueDays) override {
        if (!checkedOut) {
            cout << "This book was not checked out.\n";
            return;
        }

        checkedOut = false;

        double fine = overdueDays * 2.0;

        cout << "Book returned successfully.\n";

        if (overdueDays > 0) {
            cout << "Overdue days: " << overdueDays << endl;
            cout << "Fine: Rs. " << fixed << setprecision(2)
                 << fine << endl;
        } else {
            cout << "No fine.\n";
        }
    }

    void display() const override {
        cout << "\nType       : Book";
        cout << "\nID         : " << id;
        cout << "\nTitle      : " << title;
        cout << "\nAuthor     : " << author;
        cout << "\nStatus     : "
             << (checkedOut ? "Checked Out" : "Available")
             << endl;
    }

    string getType() const override {
        return "BOOK";
    }

    void save(ofstream& file) const override {
        file << "BOOK\n";
        file << id << "\n";
        file << title << "\n";
        file << author << "\n";
        file << checkedOut << "\n";
    }
};


// ======================================================
// DERIVED CLASS: Journal
// ======================================================

class Journal : public MediaItem {
private:
    int issueNumber;

public:
    Journal(int id, string title, int issueNumber)
        : MediaItem(id, title), issueNumber(issueNumber) {}

    void returnItem(int overdueDays) override {
        if (!checkedOut) {
            cout << "This journal was not checked out.\n";
            return;
        }

        checkedOut = false;

        // Journal fine = Rs. 1 per overdue day
        double fine = overdueDays * 1.0;

        cout << "Journal returned successfully.\n";

        if (overdueDays > 0) {
            cout << "Overdue days: " << overdueDays << endl;
            cout << "Fine: Rs. " << fixed << setprecision(2)
                 << fine << endl;
        } else {
            cout << "No fine.\n";
        }
    }

    void display() const override {
        cout << "\nType         : Journal";
        cout << "\nID           : " << id;
        cout << "\nTitle        : " << title;
        cout << "\nIssue Number : " << issueNumber;
        cout << "\nStatus       : "
             << (checkedOut ? "Checked Out" : "Available")
             << endl;
    }

    string getType() const override {
        return "JOURNAL";
    }

    void save(ofstream& file) const override {
        file << "JOURNAL\n";
        file << id << "\n";
        file << title << "\n";
        file << issueNumber << "\n";
        file << checkedOut << "\n";
    }
};


// ======================================================
// LIBRARY CLASS
// ======================================================

class Library {
private:

    // Smart pointers provide safe dynamic memory management
    vector<unique_ptr<MediaItem>> items;

public:

    // --------------------------------------------------
    // Add Book
    // --------------------------------------------------

    void addBook() {

        int id;
        string title;
        string