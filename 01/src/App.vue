<template>
  <Header />
  <div class="container">
    <Balance :total = "total"/>
    <IncomeExpenses :income="+income" :expenses="+expenses"/>
    <TransactionList :transactions="transactions" @transactionDeleted="handleTransactionDeleted" />
    <AddTransaction @transactionSubmitted="handleTransactionSubmitted"/>
  </div>
</template>

<script setup>
  import AddTransaction from './components/AddTransaction.vue';
  import Balance from './components/Balance.vue';
  import Header from './components/Header.vue';
  import IncomeExpenses from './components/IncomeExpenses.vue';
  import TransactionList from './components/TransactionList.vue';
  
  import { useToast } from 'vue-toastification';
  import { ref, computed, onMounted } from 'vue';

  const toast = useToast();
  const transactions = ref([]);

  onMounted(() => {
    const savedTransactions = JSON.parse(localStorage.getItem('transactions'));
    if (savedTransactions){
      transactions.value = savedTransactions;
    }
  });

  
  const total = computed(() => {
    return transactions.value.reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0);
  });


  const income = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount > 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0).toFixed(2);
  });


  const expenses = computed(() => {
    return transactions.value
    .filter((transaction) => transaction.amount < 0)
    .reduce((acc, transaction) => {
      return acc + transaction.amount;
    }, 0).toFixed(2);
  });

  // Add transaction 
  const handleTransactionSubmitted = (transactionData) => {
    transactions.value.push({
      id: generateUniqueId(),
      text: transactionData.text,
      amount: transactionData.amount
    });
    toast.success('Transaction added');
    saveTransactionsToLocalStorage();
  };
  

  // Generate id 
  const generateUniqueId = () =>{
    return Math.floor(Math.random() * 100000)
  };

  // Delete transaction
  const handleTransactionDeleted = (id) => {
    transactions.value = transactions.value.filter(
      (transaction) => transaction.id !== id
    );
    toast.success('Transaction deleted');
    saveTransactionsToLocalStorage();
  };
  

  // Save to localStorage
  const saveTransactionsToLocalStorage = () => {
    localStorage.setItem('transactions', JSON.stringify(transactions.value));
  }


</script>