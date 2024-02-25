# BharatIntern2
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Money Tracker</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <div class="container">
        <h2>Money Tracker</h2>
        <form id="transactionForm" action="/addTransaction" method="POST">
            <input type="text" name="description" placeholder="Description" required>
            <input type="number" name="amount" placeholder="Amount" required>
            <select name="type">
                <option value="expense">Expense</option>
                <option value="income">Income</option>
            </select>
            <button type="submit">Add Transaction</button>
        </form>
        <h3>Transactions</h3>
        <ul id="transactionList"></ul>
    </div>
    <script src="/scripts.js"></script>
</body>
</html>
/* Styles go here */
body {
    font-family: Arial, sans-serif;
    background-color: #f2f2f2;
}

.container {
    max-width: 400px;
    margin: 50px auto;
    padding: 20px;
    background-color: #fff;
    border-radius: 5px;
    box-shadow: 0px 0px 10px 0px rgba(0,0,0,0.1);
}

h2, h3 {
    text-align: center;
}

input[type="text"],
input[type="number"],
select,
button {
    display: block;
    width: 100%;
    margin-bottom: 10px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    box-sizing: border-box;
}

button {
    background-color: #4caf50;
    color: #fff;
    border: none;
    cursor: pointer;
}

button:hover {
    background-color: #45a049;
}

.transaction-expense {
    color: red;
}

.transaction-income {
    color: green;
}
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/moneyTrackerDB', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});
mongoose.connection.on('error', console.error.bind(console, 'MongoDB connection error:'));

// Create mongoose schema
const transactionSchema = new mongoose.Schema({
    description: String,
    amount: Number,
    type: String // 'expense' or 'income'
});

const Transaction = mongoose.model('Transaction', transactionSchema);

app.use(bodyParser.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));

// Route to serve the money tracker app
app.get('/', (req, res) => {
    res.sendFile(__dirname + '/index.html');
});

// Route to handle adding a new transaction
app.post('/addTransaction', (req, res) => {
    const newTransaction = new Transaction({
        description: req.body.description,
        amount: req.body.amount,
        type: req.body.type
    });

    newTransaction.save((err) => {
        if (err) {
            console.error(err);
            res.status(500).send('Error adding transaction');
        } else {
            res.redirect('/');
        }
    });
});

// Route to get all transactions
app.get('/transactions', (req, res) => {
    Transaction.find({}, (err, transactions) => {
        if (err) {
            console.error(err);
            res.status(500).send('Error retrieving transactions');
        } else {
            res.json(transactions);
        }
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
