# budget-planner
A simple Vue.js application for tracking expenses

## Project setup
```
npm install
```

### Starting the Application
```
1. npm run serve
   a. This compiles and hot-reloads for development
2. npm run mock-api
   a. This starts json-server API mocking for reading/writing data in JSON file (runs on localhost:3000)
3. http://localhost:8080 to view the app
```

### Application Flow
1. On page load, the App displays existing transactions for the current month
2. User can add transactions by clicking the + button on the right side of the screen. 
   a. This will add to the JSON file that serves as the mock API
   b. Date, Amount, and Category are all required
3. User can delete a transaction on the page by clicking the trash can icon.
   a. This will remove the transaction from the UI table and delete it from the mock API JSON file
4. User can edit a transaction on the page by clicking the pencil icon.
   a. Saving changes will update the UI table and update the data in the mock API JSON file
   b. Date, Amount, and Category are all required when modifying
5. User can select a different Month / Year in the 'Transactions For' input field at the top of the page to veiw transactions for a different month
