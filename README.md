# final-database-assignment

Library Management Data system
DROP TABLE IF EXISTS Loans;
DROP TABLE IF EXISTS Book_Authors;
DROP TABLE IF EXISTS Books;
DROP TABLE IF EXISTS Authors;
DROP TABLE IF EXISTS Publishers;
DROP TABLE IF EXISTS Borrowers;

-- -----------------------------------------------------------------------------
-- Table: Publishers
-- Description: Stores information about book publishers.
-- Relationship: One Publisher can publish many Books (1-M with Books table).
-- -----------------------------------------------------------------------------
CREATE TABLE Publishers (
    publisher_id INT AUTO_INCREMENT PRIMARY KEY, -- Unique identifier for the publisher (Primary Key)
    publisher_name VARCHAR(255) NOT NULL UNIQUE -- Name of the publisher (must be unique and not null)
);

-- -----------------------------------------------------------------------------
-- Table: Authors
-- Description: Stores information about book authors.
-- Relationship: One Author can write many Books, and one Book can have many Authors (M-M with Books table via Book_Authors).
-- -----------------------------------------------------------------------------
CREATE TABLE Authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,   -- Unique identifier for the author (Primary Key)
    first_name VARCHAR(100) NOT NULL,           -- Author's first name (must not be null)
    last_name VARCHAR(100) NOT NULL             -- Author's last name (must not be null)
    -- Consider adding UNIQUE constraint on (first_name, last_name) if names are expected to be unique,
    -- but allowing duplicates is safer for common names.
);

-- -----------------------------------------------------------------------------
-- Table: Books
-- Description: Stores information about the books in the library.
-- Relationship: Many Books can be published by one Publisher (M-1 with Publishers table).
-- Relationship: Many Books can be written by many Authors (M-M with Authors table via Book_Authors).
-- Relationship: One Book can have many Loans (1-M with Loans table).
-- -----------------------------------------------------------------------------
CREATE TABLE Books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,      -- Unique identifier for the book (Primary Key)
    title VARCHAR(255) NOT NULL,                 -- Title of the book (must not be null)
    ISBN VARCHAR(13) UNIQUE NOT NULL,            -- International Standard Book Number (must be unique and not null)
    publication_year YEAR,                       -- Year the book was published (using YEAR data type)
    publisher_id INT,                            -- Foreign Key linking to the Publishers table
    -- Add more relevant fields like genre, number_of_pages, etc.

    FOREIGN KEY (publisher_id) REFERENCES Publishers(publisher_id)
        ON DELETE SET NULL -- If a publisher is deleted, set publisher_id in Books to NULL
        ON UPDATE CASCADE   -- If a publisher_id is updated, cascade the change to Books
);

-- -----------------------------------------------------------------------------
-- Table: Book_Authors
-- Description: Linking table to handle the Many-to-Many relationship between Books and Authors.
-- Relationship: A Book can have multiple Authors, and an Author can write multiple Books.
-- -----------------------------------------------------------------------------
CREATE TABLE Book_Authors (
    book_id INT,     -- Foreign Key referencing the Books table
    author_id INT,   -- Foreign Key referencing the Authors table

    -- The combination of book_id and author_id uniquely identifies each link
    PRIMARY KEY (book_id, author_id),

    FOREIGN KEY (book_id) REFERENCES Books(book_id)
        ON DELETE CASCADE -- If a book is deleted, remove its entries in Book_Authors
        ON UPDATE CASCADE,  -- If a book_id is updated, cascade the change

    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
        ON DELETE CASCADE -- If an author is deleted, remove their entries in Book_Authors
        ON UPDATE CASCADE   -- If an author_id is updated, cascade the change
);

-- -----------------------------------------------------------------------------
-- Table: Borrowers
-- Description: Stores information about library members who borrow books.
-- Relationship: One Borrower can have many Loans (1-M with Loans table).
-- -----------------------------------------------------------------------------
CREATE TABLE Borrowers (
    borrower_id INT AUTO_INCREMENT PRIMARY KEY, -- Unique identifier for the borrower (Primary Key)
    first_name VARCHAR(100) NOT NULL,           -- Borrower's first name (must not be null)
    last_name VARCHAR(100) NOT NULL,            -- Borrower's last name (must not be null)
    email VARCHAR(255) UNIQUE,                  -- Borrower's email address (must be unique, can be null if not provided)
    phone_number VARCHAR(20),                   -- Borrower's phone number (can be null)
    address VARCHAR(255)                        -- Borrower's address (can be null)
    -- Consider adding NOT NULL to email or phone if one contact method is required
);

-- -----------------------------------------------------------------------------
-- Table: Loans
-- Description: Tracks individual book loans to borrowers.
-- Relationship: Many Loans are for one Book (M-1 with Books table).
-- Relationship: Many Loans are by one Borrower (M-1 with Borrowers table).
-- -----------------------------------------------------------------------------
CREATE TABLE Loans (
    loan_id INT AUTO_INCREMENT PRIMARY KEY,     -- Unique identifier for the loan (Primary Key)
    book_id INT NOT NULL,                       -- Foreign Key referencing the Books table (must not be null)
    borrower_id INT NOT NULL,                   -- Foreign Key referencing the Borrowers table (must not be null)
    loan_date DATE NOT NULL,                    -- Date the book was borrowed (must not be null)
    due_date DATE NOT NULL,                     -- Date the book is due (must not be null)
    return_date DATE NULL,                      -- Date the book was returned (can be null if not yet returned)
    -- Consider adding a CHECK constraint to ensure return_date is >= loan_date

    FOREIGN KEY (book_id) REFERENCES Books(book_id)
        ON DELETE RESTRICT -- Prevent deleting a book if there are active loans (or CASCADE if loans should be deleted)
        ON UPDATE CASCADE,  -- If a book_id is updated, cascade the change

    FOREIGN KEY (borrower_id) REFERENCES Borrowers(borrower_id)
        ON DELETE RESTRICT -- Prevent deleting a borrower if they have active loans (or CASCADE if loans should be deleted)
        ON UPDATE CASCADE   -- If a borrower_id is updated, cascade the change
);
