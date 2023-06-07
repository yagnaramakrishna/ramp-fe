# Instructions

Welcome to Ramp's frontend interview challenge.

In this challenge, you will need to fix certain bugs within the starter code provided to you.

The bugs **do not depend on each other**, so you can solve them independently.

You will submit a CodeSandbox link with your response.

### Prerequisites

- `node`
- `npm` or `yarn`
- [CodeSandbox](https://codesandbox.io)

### Coding

Since you need to submit a CodeSandbox link with your response (_See [Submission](#submission)_), we recommend that you create the CodeSandbox first, solve the bugs in your generated CodeSandbox, and then share the link with us. _You can also work locally first, and upload at the end._

#### Upload the project to CodeSandbox

**NOTE: Make sure your CodeSandbox link can be edited. If your link is Read only, you will be disqualified as we are not able to grade your assignment. We strongly recommend you use this method to upload your project (with the CLI) rather than importing directly from Github to generate a CodeSandbox, as it makes it Read only by default.**

- Run `yarn install` or `npm install`
- Run `yarn upload` or `npm run upload`
- If this is the first time using CodeSandbox CLI, it will ask you to log in with Github first
- You might be prompted: **We will upload XXX static files to your CodeSandbox upload storage** and then a list of files (typically `DS_Store` or `desktop.ini` files). It's fine if you upload with these, or you can manually remove them before uploading.
- Confirm that you want to proceed with deployment
- Once it finishes, you will get the link for your CodeSandbox. Also, you can log in to the website with your Github account and see your projects to retrieve the link.
- Start working directly on the CodeSandbox

_Reference: https://codesandbox.io/docs/importing#import-local-projects-via-cli_

Or

#### Run the server locally

- Run `yarn install` or `npm install`
- Run `yarn start`
- The server will be available in `http://localhost:3000`

### Special considerations

#### Typescript

At Ramp, we use React + Typescript in our codebase.

You are not required to know Typescript and using it in this challenge is optional. We have abstracted most of the Typescript code into its own files (_types.ts_), so feel free to ignore those. All of the bugs can be solved without Typescript.

If you work on the CodeSandbox, you can ignore any warnings on the code as long it works in the browser. However, feel free to write any Typescript code if your feel comfortable.

If you work locally, `TSC_COMPLE_ON_ERROR` flag is set to `true` by default. However, if you feel comfortable with Typescript, feel free to remove it on `.env` and to write any Typescript code.

#### API

We don't have a real API for this challenge, so we added some utilities to simulate API requests. To help you debug, we `console.log` the status of the ongoing simulated requests. You will not be able to see these requests in the network tab of your browser.

#### Solution

- Solutions can be HTML, CSS or Javascript oriented, depending on the bug and your solution.
- Modify any file inside the `src` folder as long as the expected result is correct.
- The goal is to solve the bug as expected. Finding a clean and efficient solution is a nice to have, but not required.
- Except for the last one, the first bugs don't depend on each other and can be solved in any order.
  - We recommend reading all the descriptions first. You might find the solution to one bug while trying to fix another.
  - The last bug will need other bugs to be fixed first in order to be reproduced.
- You cannot add any external dependency to the project. The bugs can be solved with vanilla HTML, CSS and Javascript.

---

# Bug 1: Select dropdown doesn't scroll with rest of the page

**How to reproduce:**

1. Make your viewport smaller in height. Small enough to have a scroll bar
2. Click on the **Filter by employee** select to open the options dropdown
3. Scroll down the page

**Expected:** Options dropdown moves with its parent input as you scroll the page

**Actual:** Options dropdown stays in the same position as you scroll the page, losing the reference to the select input

**My Solution** Options dropdown loses reference to the select input because the style in components/InputSelect/indedx.tsx 's RampInputSelect--dropdown-container uses the input selection position in px. When the window is resized, the input selection changes but the position input for the dropdown container remains the same so it is out of place. The fix is to replace the top and left attribute with a position: absolute which mains a reference with its parent RampInputSelect. Using relative instead of absolute works as well to maintain the correct position but also pushes the transactions downwards so the dropdown menu doesn't block anything. As the functionality remains the same, I think absolute and relative are both valid and the choice of using which comes down to design and user interaction decisions. I would also like to address the warning where 'dropdownPosition' is assigned a value but never used. My solution does not use dropdownPosition as I found using dropdownPosition with position: 'absolute' to be a fix to the bug above but presents another bug. This bug is one where the menu is skewed when resizing the window from a smaller window back to a bigger one without a scroll bar. Therefore, position: 'absolute' without the usage of dropdownPosition produced the best and bugfree result.

# Bug 2: Approve checkbox not working

**How to reproduce:**

1. Click on the checkbox on the right of any transaction

**Expected:** Clicking the checkbox toggles its value

**Actual:** Nothing happens

**My Solution** While trying to find the bug, I notice how the onChange attribute for the checkbox input wasn't activating on click. I used the inspect tool on the checkbox which led me to notice how the label is being clicked on which can be found in components/InputCheckBox/index.tsx. The label was being clicked on and not the checkbox input so the solution was to place the checkbox input within the label tag.

# Bug 3: Cannot select _All Employees_ after selecting an employee

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list
3. Click on the **Filter by employee** select to open the options dropdown
4. Select **All Employees** option

**Expected:** All transactions are loaded

**Actual:** The page crashes

**Solution** When I reproduced the bug, I was given a message about how a valid employee's id was needed for loadTransactionsByEmployee which was called in App.tsx. When all employees are selected, the id is an empty string so to fix this bug I checked for when the newValue id is an empty string. If it is, then we need to return all employees which was done by calling loadAllTransactions instead of loadTransactionsByEmployee.

# Bug 4: Clicking on View More button not showing correct data

**How to reproduce:**

1. Click on the **View more** button
2. Wait until the new data loads

**Expected:** Initial transactions plus new transactions are shown on the page

**Actual:** New transactions replace initial transactions, losing initial transactions

**Solution** The bug is that old transactions are lost when viewing more because instead of viewing a range of pages, it is instead viewing solely the next page. In other words, it is viewing only one page which is the next page. I traced the interaction to getTransactionsPaginated which was found in utils/requests.ts. The solution is to set the start page to a constant 0 and update the value of end page to page * TRANSACTIONS_PER_PAGE + TRANSACTIONS_PER_PAGE on line 27 and 28. This would allow all transaction from page 0 to the end to be viewed on view more click.

# Bug 5: Employees filter not available during loading more data

_This bug has 2 wrong behaviors that will be fixed with the same solution_

##### Part 1

**How to reproduce:**

1. Open devtools to watch the simulated network requests in the console
2. Refresh the page
3. Quickly click on the **Filter by employee** select to open the options dropdown

**Expected:** The filter should stop showing "Loading employees.." as soon as the request for `employees` is succeeded

**Actual:** The filter stops showing "Loading employees.." until `paginatedTransactions` is succeeded

##### Part 2

**How to reproduce:**

1. Open devtools to watch the simulated network requests in the console
2. Click on **View more** button
3. Quickly click on the **Filter by employee** select to open the options dropdown

**Expected:** The employees filter should not show "Loading employees..." after clicking **View more**, as employees are already loaded

**Actual:** The employees filter shows "Loading employees..." after clicking **View more** until new transactions are loaded.

**Solution** The bug here is that the filter still displays loading despite request for employees being completed. This is because it is waiting on other requests to be made which are not essential to the dropdown menu where the loading is shown. The loading can be found in loadAllTransactions in App.tsx. The solution is to move await paginatedTransactionsUtils.fetchAll() to after setIsLoading(false). once this is done, loading does not wait for paginatedTransactionsUtils.fetchAll() to finish to be set to false. It should (and hopefully is) only set to false once await employeeUtils.fetchAll() is completed.

# Bug 6: View more button not working as expected

_This bug has 2 wrong behaviors that can be fixed with the same solution. It's acceptable to fix with separate solutions as well._

##### Part 1

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list
3. Wait until transactions load

**Expected:** The **View more** button is not be visible when transactions are filtered by user, because that is not a paginated request.

**Actual:** The **View more** button is visible even when transactions are filtered by employee. _You can even click **View more** button and get an unexpected result_

##### Part 2

**How to reproduce:**

1. Click on **View more** button
2. Wait until it loads more data
3. Repeat these steps as many times as you can

**Expected:** When you reach the end of the data, the **View More** button disappears and you are not able to request more data.

**Actual:** When you reach the end of the data, the **View More** button is still showing and you are still able to click the button. If you click it, the page crashes.

**Solution** The bug here seems to be that the view more button is visible and clickable when it shouldn't be which leads to unexpected result. There shouldn't be a view more button if there are no more pages to be viewed or when transactions are filtered by employee (all transactions by employee are shown already and nothing more can be viewed). The fix would then be to remove the view more button in these situations. The solution can be found in App.tsx on line 85 where I added the checks for paginatedTransactions !== null && paginatedTransactions.nextPage !== null. This means that view more should appear only in paginated transactions and not specific employee's transactions as well as when there exist a next page.

# Bug 7: Approving a transaction won't persist the new value

_You need to fix some of the previous bugs in order to reproduce_

**How to reproduce:**

1. Click on the **Filter by employee** select to open the options dropdown
2. Select an employee from the list _(E.g. James Smith)_
3. Toggle the first transaction _(E.g. Uncheck Social Media Ads Inc)_
4. Click on the **Filter by employee** select to open the options dropdown
5. Select **All Employees** option
6. Verify values
7. Click on the **Filter by employee** select to open the options dropdown
8. Verify values

**Expected:** In steps 6 and 8, toggled transaction kept the same value it was given in step 2 _(E.g. Social Media Ads Inc is unchecked)_

**Actual:** In steps 6 and 8, toggled transaction lost the value given in step 2. _(E.g. Social Media Ads Inc is checked again)_

**Solution** In tracing this bug, I noticed how a request was made initially to load all transactions as well as clicking on filter by a specific employee for the first time. Revisiting the all transactions or an employee does not send a request. Tracing this interaction led me to usePaginatedTransactions.ts and useTransactionsByEmployee.ts in hooks. The fetchAll and fetchById functions in these files uses the fetchWithCache function from useCustomFetch.ts in hooks. The bug here is that we are fetching with a cache or saved data. Therefore, when we checked a box even though it updates the transaction, the cache or saved data is outdated and and the approved field wasn't updated. In this scenario, when we go back to all employees or a specific employee, the changes we made weren't save as it didn't change the cache. My initial thought and solution was to use fetchWithoutCache so we are fetching the correct and up to date transactions. However, this would cause unnecessary overhead or lag or longer load time as we have to wait for the data to be fetch. My better solution is to clear the cache using clearCache() when a box is checked. This is done in components/Transactions/TransactionPane.tsx on line 32. Thus, only when new changes are made, the outdated cache is cleared and rebuilt to contain the up to date data.

## Submission

**IMPORTANT:** Before sharing your CodeSandbox, open the `email.txt` file and replace your email on the only line of the file. Don't use any prefix or suffix, just your email.

You will submit a link to a CodeSandbox with your responses. Make sure your CodeSandbox is not Read only and can be edited, otherwise you will be disqualified. _See [Coding](#coding)_

---

### Callouts

- Don't remove existing `data-testid` tags. Otherwise, your results will be invalidated.
- Other than the bugs, don't modify anything that will have a different outcome. Otherwise, your results might be invalidated.
- Plagiarism is a serious offense and will result in disqualification from further consideration.
