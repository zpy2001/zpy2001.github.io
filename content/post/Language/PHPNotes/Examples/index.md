---
title: PHP - 语法样例
description: Examples about PHP
slug: language-php-examples
date: 2022-07-05 00:00:00+0000
categories:
    - Language
tags:
    - Database
    - Language
    - PHP
    - CS
weight: 1
---

# PHP note

## PHP Mysql

### connect

```php
<!-- dbconfig.php -->
<?php
    $host = 'localhost';
    $dbname = 'classicmodels';
    $username = 'root';
    $password = '';
?>

<!-- phpmysqlconnect.php -->
<?php
require_once 'dbconfig.php';

try {
    $conn = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);
    echo "Connected to $dbname at $host successfully.";
} catch (PDOException $pe) {
    die("Could not connect to the database $dbname :" . $pe->getMessage());
}
?>
```

### create table

```php
<?php

/**
 * PHP MySQL Create Table Demo
 */
class CreateTableDemo {

    /**
     * database host
     */
    const DB_HOST = 'localhost';

    /**
     * database name
     */
    const DB_NAME = 'classicmodels';

    /**
     * database user
     */
    const DB_USER = 'root';
    /*
     * database password
     */
    const DB_PASSWORD = '';

    /**
     *
     * @var type 
     */
    private $pdo = null;

    /**
     * Open the database connection
     */
    public function __construct() {
        // open database connection
        $conStr = sprintf("mysql:host=%s;dbname=%s", self::DB_HOST, self::DB_NAME);
        try {
            $this->pdo = new PDO($conStr, self::DB_USER, self::DB_PASSWORD);
        } catch (PDOException $e) {
            echo $e->getMessage();
        }
    }

    /**
     * close the database connection
     */
    public function __destruct() {
        // close the database connection
        $this->pdo = null;
    }

    /**
     * create the tasks table
     * @return boolean returns true on success or false on failure
     */
    public function createTaskTable() {
        $sql = <<<EOSQL
            CREATE TABLE IF NOT EXISTS tasks (
                task_id     INT AUTO_INCREMENT PRIMARY KEY,
                subject     VARCHAR (255)        DEFAULT NULL,
                start_date  DATE                 DEFAULT NULL,
                end_date    DATE                 DEFAULT NULL,
                description VARCHAR (400)        DEFAULT NULL
            );
EOSQL;
        return $this->pdo->exec($sql);
    }
}

// create the tasks table
$obj = new CreateTableDemo();
$obj->createTaskTable();

?>
```

### query

1. connect

```php
$pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);
```

2. sql

```php
$sql = 'SELECT lastname, firstname, jobtitle 
        FROM employees
        ORDER BY lastname';

$q = $pdo->query($sql);
```

3.setFetchMode

```php
$q->setFetchMode(PDO::FETCH_ASSOC);
```

4. 使用

```php
<?php while ($row = $q->fetch()): ?>
    <?php echo htmlspecialchars($row['lastname']) ?>
    <?php echo htmlspecialchars($row['firstname']) ?>
    <?php echo htmlspecialchars($row['jobtitle']) ?>
<?php endwhile; ?>
```

example:

```php
<?php
require_once 'dbconfig.php';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);

    $sql = 'SELECT lastname,
                    firstname,
                    jobtitle
               FROM employees
              ORDER BY lastname';

    $q = $pdo->query($sql);
    $q->setFetchMode(PDO::FETCH_ASSOC);
} catch (PDOException $e) {
    die("Could not connect to the database $dbname :" . $e->getMessage());
}
?>
<!DOCTYPE html>
<html>
    <head>
        <title>PHP MySQL Query Data Demo</title>
        <link href="css/bootstrap.min.css" rel="stylesheet">
        <link href="css/style.css" rel="stylesheet">
    </head>
    <body>
        <div id="container">
            <h1>Employees</h1>
            <table class="table table-bordered table-condensed">
                <thead>
                    <tr>
                        <th>First Name</th>
                        <th>Last Name</th>
                        <th>Job Title</th>
                    </tr>
                </thead>
                <tbody>
                    <?php while ($row = $q->fetch()): ?>
                        <tr>
                            <td><?php echo htmlspecialchars($row['lastname']) ?></td>
                            <td><?php echo htmlspecialchars($row['firstname']); ?></td>
                            <td><?php echo htmlspecialchars($row['jobtitle']); ?></td>
                        </tr>
                    <?php endwhile; ?>
                </tbody>
            </table>
    </body>
</div>
</html>
```

### insert

```php
<?php

class InsertDataDemo {

    const DB_HOST = 'localhost';
    const DB_NAME = 'classicmodels';
    const DB_USER = 'root';
    const DB_PASSWORD = '';

    private $pdo = null;

    /*
     * Open the database connection
     */
    public function __construct() {
        // open database connection
        $conStr = sprintf("mysql:host=%s;dbname=%s", self::DB_HOST, self::DB_NAME);
        try {
            $this->pdo = new PDO($conStr, self::DB_USER, self::DB_PASSWORD);
        } catch (PDOException $pe) {
            die($pe->getMessage());
        }
    }
       /**
     * Insert a row into a table
     * @return
     */
    public function insert() {
        $sql = "INSERT INTO tasks (
                      subject,
                      description,
                      start_date,
                      end_date
                  )
                  VALUES (
                      'Learn PHP MySQL Insert Dat',
                      'PHP MySQL Insert data into a table',
                      '2013-01-01',
                      '2013-01-01'
                  )";

        return $this->pdo->exec($sql);
    }

    /*
     * Insert a new task into the tasks table
     * @param string $subject
     * @param string $description
     * @param string $startDate
     * @param string $endDate
     * @return mixed returns false on failure 
     */
    function insertSingleRow($subject, $description, $startDate, $endDate) {
        $task = array(':subject' => $subject,
            ':description' => $description,
            ':start_date' => $startDate,
            ':end_date' => $endDate);

        $sql = 'INSERT INTO tasks (
                      subject,
                      description,
                      start_date,
                      end_date
                  )
                  VALUES (
                      :subject,
                      :description,
                      :start_date,
                      :end_date
                  );';

        $q = $this->pdo->prepare($sql);

        return $q->execute($task);
    }
}

$obj = new InsertDataDemo();
$obj->insert();
$obj->insertSingleRow('MySQL PHP Insert Tutorial', 
                          'MySQL PHP Insert using prepared statement', 
                          '2013-01-01', 
                          '2013-01-02');
?>
```

### update

```php
<?php

/**
 * PHP MySQL Update data demo
 */
class UpdateDataDemo {

    const DB_HOST = 'localhost';
    const DB_NAME = 'classicmodels';
    const DB_USER = 'root';
    const DB_PASSWORD = '';

    /**
     * PDO instance
     * @var PDO
     */
    private $pdo = null;

    /**
     * Open the database connection
     */
    public function __construct() {
        // open database connection
        $connStr = sprintf("mysql:host=%s;dbname=%s", self::DB_HOST, self::DB_NAME);
        try {
            $this->pdo = new PDO($connStr, self::DB_USER, self::DB_PASSWORD);
        } catch (PDOException $e) {
            die($e->getMessage());
        }
    }

    /**
     * Update an existing task in the tasks table
     * @param string $subject
     * @param string $description
     * @param string $startDate
     * @param string $endDate
     * @return bool return true on success or false on failure
     */
    public function update($id, $subject, $description, $startDate, $endDate) {
        $task = [
            ':taskid' => $id,
            ':subject' => $subject,
            ':description' => $description,
            ':start_date' => $startDate,
            ':end_date' => $endDate];


        $sql = 'UPDATE tasks
                    SET subject      = :subject,
                         start_date  = :start_date,
                         end_date    = :end_date,
                         description = :description
                  WHERE task_id = :taskid';

        $q = $this->pdo->prepare($sql);

        return $q->execute($task);
    }

    /**
     * close the database connection
     */
    public function __destruct() {
        // close the database connection
        $this->pdo = null;
    }

}

$obj = new UpdateDataDemo();

if ($obj->update(2, 'MySQL PHP Update Tutorial', 
                    'MySQL PHP Update using prepared statement', 
                    '2013-01-01', 
                    '2013-01-01') !== false)
    echo 'The task has been updated successfully';
else
    echo 'Error updated the task';
```

### delete

```php
<?php

/**
 * PHP MySQL Delete Data Demo
 */
class DeleteDataDemo {

    const DB_HOST = 'localhost';
    const DB_NAME = 'classicmodels';
    const DB_USER = 'root';
    const DB_PASSWORD = '';

    /**
     * PDO instance
     * @var PDO 
     */
    private $pdo = null;

    /**
     * Open a database connection to MySQL
     */
    public function __construct() {
        // open database connection
        $conStr = sprintf("mysql:host=%s;dbname=%s", self::DB_HOST, self::DB_NAME);
        try {
            $this->pdo = new PDO($conStr, self::DB_USER, self::DB_PASSWORD);
        } catch (PDOException $e) {
            die($e->getMessage());
        }
    }

    /**
     * Delete a task based on a specified task id
     * @param int $id
     * @return bool true on success or false on failure
     */
    public function delete($id) {

        $sql = 'DELETE FROM tasks
                WHERE task_id = :task_id';

        $q = $this->pdo->prepare($sql);

        return $q->execute([':task_id' => $id]);
    }

    /**
    * Delete all rows in the tasks table
    */
    public function deleteAll(){
        $sql = 'DELETE FROM tasks';
        return $this->pdo->exec($sql);
    }

    public function truncateTable() {
        $sql = 'TRUNCATE TABLE tasks';
        return $this->pdo->exec($sql);
    }

    /**
     * close the database connection
     */
    public function __destruct() {
        $this->pdo = null;
    }

}

$obj = new DeleteDataDemo();
// delete id 2
$obj->delete(2);
?>
```

### call procedure

```php
$sql = 'CALL xx()';
$q = $pdo->query($sql);
$q->setFetchMode(PDO::FETCH_ASSOC);

// Calling stored procedures with an OUT parameter
// calling stored procedure command
$sql = 'CALL GetCustomerLevel(:id,@level)';

// prepare for execution of the stored procedure
$stmt = $pdo->prepare($sql);

// pass value to the command
$stmt->bindParam(':id', $customerNumber, PDO::PARAM_INT);

// execute the stored procedure
$stmt->execute();

$stmt->closeCursor();

// execute the second query to get customer's level
$row = $pdo->query("SELECT @level AS level")->fetch(PDO::FETCH_ASSOC);
```

### transanction

```php
<?php

/**
 * PHP MySQL Transaction Demo
 */
class TransactionDemo {

    const DB_HOST = 'localhost';
    const DB_NAME = 'classicmodels';
    const DB_USER = 'root';
    const DB_PASSWORD = '';

    /**
     * Open the database connection
     */
    public function __construct() {
        // open database connection
        $conStr = sprintf("mysql:host=%s;dbname=%s", self::DB_HOST, self::DB_NAME);
        try {
            $this->pdo = new PDO($conStr, self::DB_USER, self::DB_PASSWORD);
        } catch (PDOException $e) {
            die($e->getMessage());
        }
    }

    /**
     * PDO instance
     * @var PDO 
     */
    private $pdo = null;

    /**
     * Transfer money between two accounts
     * @param int $from
     * @param int $to
     * @param float $amount
     * @return true on success or false on failure.
     */
    public function transfer($from, $to, $amount) {

        try {
            $this->pdo->beginTransaction();

            // get available amount of the transferer account
            $sql = 'SELECT amount FROM accounts WHERE id=:from';
            $stmt = $this->pdo->prepare($sql);
            $stmt->execute(array(":from" => $from));
            $availableAmount = (int) $stmt->fetchColumn();
            $stmt->closeCursor();

            if ($availableAmount < $amount) {
                echo 'Insufficient amount to transfer';
                return false;
            }
            // deduct from the transferred account
            $sql_update_from = 'UPDATE accounts
				SET amount = amount - :amount
				WHERE id = :from';
            $stmt = $this->pdo->prepare($sql_update_from);
            $stmt->execute(array(":from" => $from, ":amount" => $amount));
            $stmt->closeCursor();

            // add to the receiving account
            $sql_update_to = 'UPDATE accounts
                                SET amount = amount + :amount
                                WHERE id = :to';
            $stmt = $this->pdo->prepare($sql_update_to);
            $stmt->execute(array(":to" => $to, ":amount" => $amount));

            // commit the transaction
            $this->pdo->commit();

            echo 'The amount has been transferred successfully';

            return true;
        } catch (PDOException $e) {
            $this->pdo->rollBack();
            die($e->getMessage());
        }
    }

    /**
     * close the database connection
     */
    public function __destruct() {
        // close the database connection
        $this->pdo = null;
    }

}

// test the transfer method
$obj = new TransactionDemo();

// transfer 30K from from account 1 to 2
$obj->transfer(1, 2, 30000);


// transfer 5K from from account 1 to 2
$obj->transfer(1, 2, 5000);
?>
```