<template>
  <div class="container">
    <h1>Expense Tracker</h1>
    <hr>
    <div class="row">
      <div :class="tableClass">
        <h5> 
          Transactions for <input type="month" class="form-input" v-model="displayedMonth">
          <div class="float-right"> 
            <b-icon icon="plus-square-fill" font-scale="1.5" aria-hidden="true" variant="success" @click="showInputForm(null)"/>
          </div>
        </h5>
        <b-table striped hover :items="transactions" :fields="transactionFields">
          <template v-slot:cell(actions)="row">
            <b-icon icon="trash" aria-hidden="true" @click="deleteTransaction(row.item)"></b-icon>
            <b-icon icon="pencil" aria-hidden="true" @click="showInputForm(row.item)"></b-icon>
          </template>
        </b-table>
      </div>
      <div :class="inputFormClass" v-show="inputFormVisible">
        <div v-if="submissionErrs.length">
          <b>Please correct the following error(s):</b>
          <ul>
            <li v-for="error in submissionErrs" v-bind:key="error">{{ error }}</li>
          </ul>
        </div>
        <form>
          <b-form-datepicker :max="endOfCurrMonth"
            :date-format-options="{ year: 'numeric', month: 'numeric', day: 'numeric' }"
            locale="en"
            v-model="transDateInput"></b-form-datepicker>
          <b-input-group prepend="$">
            <b-form-input v-model.number="transAmountInput" type="number" min="0"></b-form-input>
          </b-input-group>
          <b-input-group prepend="Category">
            <b-form-select v-model="transCatInput" :options="categories" />
          </b-input-group>
          <b-button class="float-right" @click="hideInputForm"> Cancel </b-button>
          <b-button class="float-right" variant="success" @click="submitTransactionData"> Save Transaction </b-button>
        </form>
      </div>
    </div>
  </div>
</template>
<script>
  import axios from 'axios'
  import moment from 'moment'
  export default {
    name: 'Budget',
    data () {
      return {
        currDate: '',
        displayedMonth: '',
        displayedMonthStart: null,
        displayedMonthEnd: null,
        submissionErrs: [],
        transAmountInput: 0,
        transCatInput: '',
        transDateInput: '',
        editingId: 0,
        inputFormVisible: false,
        categories: ['Bills', 'Transportation', 'Entertainment', 'Food', 'Miscellaneous'],
        transactions: null,
        transactionFields: [
          {
            key:'date',
            sortable: true,
            formatter: (value) => { return moment(value).format('MM/DD/YYYY') }
          }, 
          {
            key: 'amount',
            sortable: true,
            formatter: (value) => { return "$" + value.toFixed(2) }
          }, 
          {
            key: 'category',
            sortable: true
          }, 
          {
            key: 'actions',
            label: ''
          }
        ]
      }
    },
    created () {
      // on page create, set current month to trigger data load
      let currDate = moment()
      this.endOfCurrMonth = currDate.endOf('month').format('YYYY-MM-DD')
      this.displayedMonth = currDate.format("YYYY-MM")
    },
    watch: {
      // load new data when selected month changes
      displayedMonth: function(newMonth) {
        let newMonthDate = moment(newMonth)
        this.fetchTransactionsForMonth(newMonthDate)
      }
    },
    computed: {
      /*
      * determine class of input section based on whether its visible
      */
      inputFormClass () {
        if (this.inputFormVisible) {
          return "col-lg-4"
        }
        return ""
      },
      /*
      * determine class of table section based on whether form is visible
      */
      tableClass () {
        if (this.inputFormVisible) {
          return "col-lg-8"
        }
        return "col-lg-12"
      }
    },
    methods: {
      /*
      * Get all transactions that fall in the date of the provided month and add
      * them to the list of transactions displayed in the UI
      */
      fetchTransactionsForMonth (selectedDate) {
        // calculate start and end date-time for the provided date
        this.displayedMonthStart = moment(selectedDate).subtract(1,'months').endOf('month').format('YYYY-MM-DD HH:mm:ss')
        this.displayedMonthEnd = selectedDate.endOf('month').format('YYYY-MM-DD HH:mm:ss')
        
        // retrieve transactions that fall within the selected month
        axios.get('http://localhost:3000/transaction/', { params: {
          _sort: 'date',
          date_gte: this.displayedMonthStart,
          date_lte: this.displayedMonthEnd
        }})
        .then(res => {
          this.transactions = res.data
        })
        .catch(err => {
          this.transactions = null
          console.log('Unable to load data ' + err)
        })
      },
      /*
      * Create a transaction using the input fields in the UI form, and add the transaction to 
      * the UI table if it falls within the month for which data is currently being displayed
      */
      createTransaction () {
        let transaction = {
          "date": this.transDateInput,
          "amount": this.transAmountInput,
          "category": this.transCatInput
        }
        // add transaction. If transaction date is in the month currently displayed, add to table
        axios.post('http://localhost:3000/transaction/', transaction)
        .then((res) => {
          if (moment(transaction.date).isBetween(this.displayedMonthStart, this.displayedMonthEnd, null, '[]')) {
            this.transactions.push(res.data)
          }
          this.clearInputForm()
        })
        .catch(err => {
          console.log('Unable to add transaction data ' + err)
        })
      },
      /*
      * Update the details of an existing transaction via API call.
      * If transaction is still in the month displayed in UI, update details in table
      * Else remove from table
      */
      editTransaction () {
        let transaction = {
          "id": this.editingId,
          "date": this.transDateInput,
          "amount": this.transAmountInput,
          "category": this.transCatInput
        }
        axios.put('http://localhost:3000/transaction/' + transaction.id, transaction)
        .then((res) => {
          let transIndex = this.transactions.findIndex(trans => trans.id === this.editingId)
          if (transIndex !== -1) {
            if (moment(transaction.date).isBetween(this.displayedMonthStart, this.displayedMonthEnd, null, '[]')) {
              this.transactions[transIndex].date = res.data.date
              this.transactions[transIndex].amount = res.data.amount
              this.transactions[transIndex].category = res.data.category
            } else {
              this.transactions.splice(transIndex, 1)
            }
          }
          this.hideInputForm()
        })
        .catch(err => {
          console.log('Unable to add transaction data ' + err)
        })
      },
      /*
      * Delete a single transaction by ID, and remove it from the UI table
      */
      deleteTransaction (transaction) {
        // delete transaction and remove from UI table
        axios.delete('http://localhost:3000/transaction/' + transaction.id)
        .then(() => {
          let transIndex = this.transactions.indexOf(transaction)
          if (transIndex !== -1) {
            this.transactions.splice(transIndex, 1)
          }
        })
        .catch(err => {
          console.log('Unable to load data ' + err)
        })
      },
      /*
      * Checks that all fields contain data. If any are missing, errors are added to the list of
      * errors displayed in the UI
      */
      validateInput () {
        this.submissionErrs = []
        if (!this.transDateInput) {
          this.submissionErrs.push('Transaction Date is required')
        }
        if (!this.transAmountInput) {
          this.submissionErrs.push('Transaction Amount is required')
        }
        if (!this.transCatInput) {
          this.submissionErrs.push('Transaction Category is required')
        }

        if (!this.submissionErrs.length) {
          return true
        }
        return false
      },
      /*
      * Toggle flag that determines whether the input form is displayed, and determine whether
      * the form is for editing or creating based on whether an existing transaction is passed
      * in the params
      */
      showInputForm (transaction) {
        if (transaction) {
          this.transDateInput = transaction.date
          this.transAmountInput = transaction.amount
          this.transCatInput = transaction.category
          this.editingId = transaction.id
        } else {
          this.editingId = 0
        }
        this.inputFormVisible = true
      },
      /*
      * Hide input form and clear all input form fields
      */
      hideInputForm () {
        this.inputFormVisible = false
        this.clearInputForm()
      },
      /*
      * Clear input form fields and array of errors
      */
      clearInputForm () {
        // clear all inputs in form
        this.transDateInput = ''
        this.transCatInput = ''
        this.transAmountInput = ''
        this.submissionErrs = []
      },
      /*
      * Submit transaction data for persisting via API. 
      * If UI input values are missing, show message on screen.
      * Else, call create or edit bassed on the whether editingId is set to an ID value
      */
      submitTransactionData () {
        if (!this.validateInput()) {
          return
        }
        if (!this.editingId) {
          this.createTransaction()
        } else {
          this.editTransaction()
        }
      }
    }
  }
</script>
<style>
  .float-right {
    float: right !important;
  }
</style>