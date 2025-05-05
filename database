/* Topik 1 Views */
/*i. view v_customer_all menampilkan semua isi tabel customers.*/
CREATE VIEW v_customer_all AS
SELECT *
FROM customers;

SELECT * FROM v_customer_all; 

/*ii. view v_deposit_transaction menampilkan semua transaksi deposit.*/
CREATE VIEW v_deposit_transaction AS
SELECT t.*
FROM transactions t
JOIN transaction_types tt ON t.transaction_type_id = tt.transaction_type_id
WHERE tt.name = 'deposit';

SELECT * FROM v_deposit_transaction;

/*iii. view v_transfer_transaction menampilkan semua transaksi transfer.*/
CREATE VIEW v_transfer_transaction AS
SELECT t.*
FROM transactions t
JOIN transaction_types tt ON t.transaction_type_id = tt.transaction_type_id
WHERE tt.name = 'transfer';

SELECT * FROM v_transfer_transaction; 

/*Topik 2 Procedures*/
/*i. Procedure sp_CreateCustomer untuk menambahkan customer baru ke dalam tabel customers: input first_name, last_name, email, phone_number, address.*/
CREATE PROCEDURE sp_CreateCustomer
    @first_name VARCHAR(50),
    @last_name VARCHAR(50),
    @email VARCHAR(50),
    @phone_number VARCHAR(20),
    @address VARCHAR(255)
AS
BEGIN
    INSERT INTO customers (customer_id, first_name, last_name, email, phone_number, address, created_at)
    VALUES (NEWID(), @first_name, @last_name, @email, @phone_number, @address, GETDATE());
END;

EXEC sp_CreateCustomer @first_name='Alisha', @last_name='Rahman', 
@email='alisha@example.com', @phone_number='081234567890', @address='Jl. Mawar No. 10, Semarang';

SELECT * FROM customers 
WHERE first_name = 'Alisha'; 

/*ii. Procedure sp_CreateAccount untuk membuat akun baru untuk customer yang sudah ada: input customer_id, account_number, account_type, balance. Notes: Tambahkan validasi untuk memastikan bahwa customer_id benar benar ada pada table customers.*/
CREATE PROCEDURE sp_CreateAccount
    @customer_id CHAR(36),
    @account_number VARCHAR(20),
    @account_type VARCHAR(20),
    @balance DECIMAL(18, 2)
AS
BEGIN
    IF EXISTS (SELECT 1 FROM customers WHERE customer_id = @customer_id)
    BEGIN
        INSERT INTO accounts (account_id, customer_id, account_number, account_type, balance, created_at)
        VALUES (NEWID(), @customer_id, @account_number, @account_type, @balance, GETDATE());
    END
    ELSE
    BEGIN
        PRINT 'Customer ID tidak ditemukan';
    END
END;

EXEC sp_CreateAccount 
    @customer_id = '123e4567-e89b-12d3-a456-426614174000', 
    @account_number = '9876543210', 
    @account_type = 'Savings', 
    @balance = 5000000;

EXEC sp_CreateAccount 
    @customer_id = '9d78736f-df51-4622-9cb1-c4db88dca2d0', 
    @account_number = '2075461919', 
    @account_type = 'current', 
    @balance = 10838.30;

SELECT * FROM accounts 
where customer_id = '9d78736f-df51-4622-9cb1-c4db88dca2d0'; 

/*iii. Procedure sp_MakeTransaction untuk menambahkan transaksi baru: input account_id, transaction_type_id, amount, description, reference_account (opsional). Notes: Tambahkan validasi untuk transfer, jika jumlah transfer melebihi balance maka transaksi dihentikan.*/
CREATE PROCEDURE sp_MakeTransaction
    @account_id CHAR(36),
    @transaction_type_id INT,
    @amount DECIMAL(18,2),
    @description VARCHAR(255),
    @reference_account CHAR(36) = NULL
AS
BEGIN
    DECLARE @current_balance DECIMAL(18,2);
    SELECT @current_balance = balance FROM accounts WHERE account_id = @account_id;

    IF @transaction_type_id = 2 AND @amount > @current_balance 
        RETURN;

    INSERT INTO transactions (transaction_id, account_id, transaction_type_id, amount, description, reference_account)
    VALUES (NEWID(), @account_id, @transaction_type_id, @amount, @description, @reference_account);

    IF @transaction_type_id != 1 
        UPDATE accounts SET balance = balance - @amount WHERE account_id = @account_id;
END;

DECLARE @account_id CHAR(36) = '07a36a4b-7943-4634-b17a-7c7eb4407cd3';
DECLARE @transaction_type_id INT = 3;
DECLARE @amount DECIMAL(18,2) = 100;
DECLARE @description VARCHAR(255) = 'Transfer ke teman';
DECLARE @reference_account CHAR(36) = '72923966-A27D-4221-B929-B2B1CD7AA139';
EXEC sp_MakeTransaction
    @account_id,
    @transaction_type_id,
    @amount,
    @description,
    @reference_account;

SELECT * FROM transactions 
WHERE account_id = '07a36a4b-7943-4634-b17a-7c7eb4407cd3';

/*iv. Procedure sp_GetCustomerSummary untuk menampilkan ringkasan data customer berdasarkan customer_id:
1. Nama lengkap customer
2. Jumlah akun yang dimiliki
3. Jumlah total saldo semua akun
4. Jumlah pinjaman aktif
5. Total pinjaman amount aktif*/
CREATE PROCEDURE sp_GetCustomerSummary
    @customer_id CHAR(36)
AS
BEGIN
    SELECT 
        CONCAT(first_name, ' ', last_name) AS full_name,
        (SELECT COUNT(*) FROM accounts WHERE customer_id = @customer_id) AS total_accounts,
        (SELECT SUM(balance) FROM accounts WHERE customer_id = @customer_id) AS total_balance,
        (SELECT COUNT(*) FROM loans WHERE customer_id = @customer_id AND status = 'Active') AS active_loans,
        (SELECT SUM(loan_amount) FROM loans WHERE customer_id = @customer_id AND status = 'Active') AS total_loan_amount
    FROM customers
    WHERE customer_id = @customer_id;
END;


EXEC sp_GetCustomerSummary '9d78736f-df51-4622-9cb1-c4db88dca2d0';
