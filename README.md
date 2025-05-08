# Welcome to my PEI Pizza's Customer Ordering System using PL/SQL !!!

step-1

We begin by creating a **DOMINOS** table to store the information about our loyal pizza customers.
```sql
CREATE TABLE DOMINOS_CUSTOMERS (
    customer_id   NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    first_name    VARCHAR2(50) NOT NULL,
    last_name     VARCHAR2(50) NOT NULL,
    phone_number  VARCHAR2(20) UNIQUE, -- Phone number could be a good unique identifier
    join_date     DATE DEFAULT SYSDATE NOT NULL
);
```
![Alt text](1.png)
Next we will create **Dominos** table to record each order placed by a customer and define a foreign key constraint linking orders to customers.
```sql
CREATE TABLE DOMINOS_ORDERS (
    order_id         NUMBER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id      NUMBER NOT NULL,
    order_date       DATE DEFAULT SYSDATE NOT NULL,
    order_total      NUMBER(10, 2) NOT NULL,
    delivery_method  VARCHAR2(20), -- e.g., 'Delivery', 'Carryout'
    order_status     VARCHAR2(20) DEFAULT 'Pending' NOT NULL, -- e.g., 'Pending', 'Complete'

    CONSTRAINT fk_dominos_customer
        FOREIGN KEY (customer_id)
        REFERENCES DOMINOS_CUSTOMERS(customer_id)
);
```
![Alt text](2.png)

step - 2

procedure creation

```sql
CREATE OR REPLACE PROCEDURE ADD_DOMINOS_CUSTOMER (
    p_first_name   IN DOMINOS_CUSTOMERS.first_name%TYPE,
    p_last_name    IN DOMINOS_CUSTOMERS.last_name%TYPE,
    p_phone_number IN DOMINOS_CUSTOMERS.phone_number%TYPE
)
IS
BEGIN
    INSERT INTO DOMINOS_CUSTOMERS (first_name, last_name, phone_number)
    VALUES (p_first_name, p_last_name, p_phone_number);

    DBMS_OUTPUT.PUT_LINE('Customer ' || p_first_name || ' ' || p_last_name || ' added.');

END ADD_DOMINOS_CUSTOMER;
/
```
message after executing this block: 'Procedure ADD_DOMINOS_CUSTOMER compiled'

to compile:
```sql
SET SERVEROUTPUT ON;
BEGIN 
  ADD_DOMINOS_CUSTOMER('John', 'Wick', '555-1234'); ADD_DOMINOS_CUSTOMER('Jane', 'Mary', '555-5678'); 
END;
/
```

To verify the data:
```sql
SELECT * FROM DOMINOS_CUSTOMERS;
```
![Alt text](3.png)

step- 3
handling orders

```sql
CREATE OR REPLACE PROCEDURE CREATE_DOMINOS_ORDER (
    p_customer_id      IN DOMINOS_CUSTOMERS.customer_id%TYPE,
    p_order_total      IN DOMINOS_ORDERS.order_total%TYPE,
    p_delivery_method  IN DOMINOS_ORDERS.delivery_method%TYPE,
    p_order_status     IN DOMINOS_ORDERS.order_status%TYPE DEFAULT 'Pending' 
)
IS
    v_customer_count NUMBER;
BEGIN
    SELECT COUNT(*)
    INTO v_customer_count
    FROM DOMINOS_CUSTOMERS
    WHERE customer_id = p_customer_id;

    IF v_customer_count = 1 THEN
        INSERT INTO DOMINOS_ORDERS (customer_id, order_total, delivery_method, order_status)
        VALUES (p_customer_id, p_order_total, p_delivery_method, p_order_status);

        DBMS_OUTPUT.PUT_LINE('Order created successfully for customer ID ' || p_customer_id);

    ELSE
        DBMS_OUTPUT.PUT_LINE('Error: Customer with ID ' || p_customer_id || ' not found. Order not created.');
    END IF;

END CREATE_DOMINOS_ORDER;
/
```
compile and test

```sql
SET SERVEROUTPUT ON;
BEGIN
    CREATE_DOMINOS_ORDER(
        p_customer_id => 1,
        p_order_total => 25.50,
        p_delivery_method => 'Delivery'
    );

     CREATE_DOMINOS_ORDER(
        p_customer_id => 2,
        p_order_total => 15.00,
        p_delivery_method => 'Carryout',
        p_order_status => 'Complete' 
    );
    DBMS_OUTPUT.PUT_LINE('Transaction committed.');
END;
/
```
we can see our two customers loves pizza!!
![Alt text](4.png)

step - 4
custom functions

```sql
CREATE OR REPLACE FUNCTION GET_CUSTOMER_ORDER_COUNT (
    p_customer_id IN DOMINOS_CUSTOMERS.customer_id%TYPE
)
RETURN NUMBER 
IS
    v_order_count NUMBER := 0; 
BEGIN
    SELECT COUNT(*)
    INTO v_order_count
    FROM DOMINOS_ORDERS
    WHERE customer_id = p_customer_id;

    RETURN v_order_count;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;

END GET_CUSTOMER_ORDER_COUNT;
/
```

compiling
```sql
SET SERVEROUTPUT ON;
DECLARE
    v_customer_id_to_check NUMBER := 1; 
    v_orders_count NUMBER;
BEGIN
    v_orders_count := GET_CUSTOMER_ORDER_COUNT(v_customer_id_to_check);
    DBMS_OUTPUT.PUT_LINE('Customer ID ' || v_customer_id_to_check || ' has ' || v_orders_count || ' orders.');

    DBMS_OUTPUT.PUT_LINE('Customer ID 2 has ' || GET_CUSTOMER_ORDER_COUNT(2) || ' orders.'); 
    DBMS_OUTPUT.PUT_LINE('Customer ID 999 has ' || GET_CUSTOMER_ORDER_COUNT(3) || ' orders.'); 
END;
/
```
![Alt text](5.png)

STEP - 4

procedure utilizing IN OUT parameter. 
```sql
CREATE OR REPLACE PROCEDURE APPLY_DISCOUNT (
    p_order_total   IN OUT NUMBER, 
    p_discount_rate IN NUMBER      
)
IS
BEGIN
    p_order_total := p_order_total * (1 - p_discount_rate);
END APPLY_DISCOUNT;
/
```

compiling
```sql
SET SERVEROUTPUT ON;

DECLARE
    v_current_total NUMBER := 100.00; 
    v_discount_rate NUMBER := 0.10;   
BEGIN
    DBMS_OUTPUT.PUT_LINE('Original Total: ' || v_current_total);

    APPLY_DISCOUNT(
        p_order_total   => v_current_total, 
        p_discount_rate => v_discount_rate
    );

    DBMS_OUTPUT.PUT_LINE('Total After Discount: ' || v_current_total);

END;
/
```
![Alt text](6.png)

















