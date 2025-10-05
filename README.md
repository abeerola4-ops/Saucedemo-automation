
## File Structure:
```
saucedemo-automation/
├── package.json
├── playwright.config.js
├── .eslintrc.js
├── .prettierrc
├── README.md
├── tests/
│   └── e2e/
│       └── purchase-flow.spec.js
├── pages/
│   ├── LoginPage.js
│   ├── InventoryPage.js
│   ├── CartPage.js
│   ├── CheckoutPage.js
│   └── ConfirmationPage.js
├── data/
│   └── test-data.json
├── utils/
│   └── helpers.js
└── .github/
    └── workflows/
        └── playwright-tests.yml
```

## 1. package.json
```json
{
  "name": "saucedemo-automation",
  "version": "1.0.0",
  "description": "Test automation for SauceDemo website",
  "scripts": {
    "test": "npx playwright test",
    "test:headed": "npx playwright test --headed",
    "test:debug": "npx playwright test --debug",
    "test:smoke": "npx playwright test --grep @smoke",
    "test:regression": "npx playwright test --grep @regression",
    "test:chrome": "npx playwright test --project=chromium",
    "test:firefox": "npx playwright test --project=firefox",
    "test:webkit": "npx playwright test --project=webkit",
    "report": "npx playwright show-report",
    "lint": "eslint tests/ pages/ utils/",
    "lint:fix": "eslint tests/ pages/ utils/ --fix",
    "format": "prettier --write tests/ pages/ utils/"
  },
  "devDependencies": {
    "@playwright/test": "^1.40.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  },
  "engines": {
    "node": ">=18"
  }
}
```

## 2. playwright.config.js
```javascript
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 1,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { outputFolder: 'reports/html-report' }],
    ['json', { outputFolder: 'reports/json-report' }]
  ],
  use: {
    baseURL: 'https://www.saucedemo.com',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    }
  ],
  outputDir: 'test-results/',
});
```

## 3. .eslintrc.js
```javascript
module.exports = {
  env: {
    node: true,
    es2021: true,
  },
  extends: ['eslint:recommended'],
  parserOptions: {
    ecmaVersion: 12,
    sourceType: 'module',
  },
  rules: {
    'no-unused-vars': 'warn',
    'no-console': 'off',
    'prefer-const': 'error',
    'no-var': 'error',
  },
};
```

## 4. .prettierrc
```json
{
  "semi": true,
  "trailingComma": "none",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false
}
```

## 5. README.md
```markdown
# SauceDemo Test Automation

This project contains automated tests for the SauceDemo website using Playwright.

## What's included

- Page Object Model pattern implementation
- Multi-browser testing support
- Parallel test execution
- Failure screenshots
- HTML test reports
- GitHub Actions CI setup
- Code quality tools

## Project layout

```
saucedemo-automation/
├── tests/e2e/          # Test files
├── pages/              # Page classes
├── data/               # Test data
├── utils/              # Utilities
├── screenshots/        # Failure images
├── reports/            # Test reports
└── .github/workflows/  # CI configuration
```

## Getting started

1. Get the code
   ```bash
   git clone <repo-url>
   cd saucedemo-automation
   ```

2. Install required packages
   ```bash
   npm install
   ```

3. Install browsers
   ```bash
   npx playwright install
   ```

## Running tests

Run all tests
```bash
npm test
```

Run with browser UI
```bash
npm run test:headed
```

Run specific test groups
```bash
npm run test:smoke        # Smoke tests
npm run test:regression   # Regression tests
npm run test:chrome       # Chrome tests
```

## Viewing results

Open HTML report after tests complete:
```bash
npm run report
```

## CI setup

GitHub Actions runs automatically on:
- Push to main/develop branches
- Pull requests to main
- Includes code style checks
- Runs regression tests

## Technologies

- Playwright: Testing framework
- JavaScript: Programming language
- ESLint: Code linting
- Prettier: Code formatting
- GitHub Actions: Continuous integration

## Notes

- Tests require SauceDemo website access
- Test data assumes consistent website state
- Browsers need JavaScript support

## Known constraints

- External website dependency
- UI-only testing approach
```

## 6. tests/e2e/purchase-flow.spec.js
```javascript
const { test, expect } = require('@playwright/test');
const LoginPage = require('../../pages/LoginPage');
const InventoryPage = require('../../pages/InventoryPage');
const CartPage = require('../../pages/CartPage');
const CheckoutPage = require('../../pages/CheckoutPage');
const ConfirmationPage = require('../../pages/ConfirmationPage');
const testData = require('../../data/test-data.json');

test.describe('SauceDemo Purchase Tests', () => {
  let loginPage, inventoryPage, cartPage, checkoutPage, confirmationPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    inventoryPage = new InventoryPage(page);
    cartPage = new CartPage(page);
    checkoutPage = new CheckoutPage(page);
    confirmationPage = new ConfirmationPage(page);
  });

  test('@smoke Complete purchase process with valid login', async ({ page }) => {
    await loginPage.goToLoginPage();
    await loginPage.loginWithValidUser(testData.users);
    await inventoryPage.verifyPageLoaded();

    await inventoryPage.changeSortOrder('lohi');
    const selectedProducts = await inventoryPage.addLowCostItemsToCart(2);
    
    await inventoryPage.openCart();
    await cartPage.verifyPageLoaded();
    await cartPage.checkCartContents(selectedProducts);
    
    await cartPage.startCheckout();
    await checkoutPage.verifyPageLoaded();
    await checkoutPage.enterCustomerDetails(testData.customer);
    await checkoutPage.goToOverview();
    
    await confirmationPage.verifyPageLoaded();
    await confirmationPage.verifyPricing(selectedProducts);
    await confirmationPage.finalizeOrder();
    
    const message = await confirmationPage.getConfirmationMessage();
    expect(message).toBe('Thank you for your order!');
  });

  test('@regression Test login with wrong credentials', async ({ page }) => {
    await loginPage.goToLoginPage();
    await loginPage.loginWithInvalidUser();
    
    const error = await loginPage.getErrorText();
    expect(error).toBe(testData.errorMessages.invalidLogin);
  });

  test('@regression Check cart items and pricing', async ({ page }) => {
    await loginPage.goToLoginPage();
    await loginPage.loginWithValidUser(testData.users);
    await inventoryPage.verifyPageLoaded();

    await inventoryPage.changeSortOrder('lohi');
    const selectedProducts = await inventoryPage.addLowCostItemsToCart(2);
    
    const itemCount = await inventoryPage.getCartQuantity();
    expect(itemCount).toBe(2);
    
    await inventoryPage.openCart();
    await cartPage.verifyPageLoaded();
    
    const cartContents = await cartPage.getItemsInCart();
    expect(cartContents).toHaveLength(2);
    
    for (let i = 0; i < selectedProducts.length; i++) {
      expect(cartContents[i].name).toBe(selectedProducts[i].name);
      expect(cartContents[i].price).toBe(selectedProducts[i].price);
    }
  });
});
```

## 7. pages/LoginPage.js
```javascript
const { expect } = require('@playwright/test');

class LoginPage {
  constructor(page) {
    this.page = page;
    this.usernameField = '[data-test="username"]';
    this.passwordField = '[data-test="password"]';
    this.loginBtn = '[data-test="login-button"]';
    this.errorContainer = '[data-test="error"]';
  }

  async goToLoginPage() {
    await this.page.goto('/');
    await expect(this.page.locator(this.loginBtn)).toBeVisible();
  }

  async enterCredentials(username, password) {
    await this.page.fill(this.usernameField, username);
    await this.page.fill(this.passwordField, password);
  }

  async clickLogin() {
    await this.page.click(this.loginBtn);
  }

  async getErrorText() {
    return await this.page.textContent(this.errorContainer);
  }

  async loginWithValidUser(userData) {
    await this.enterCredentials(userData.standard.username, userData.standard.password);
    await this.clickLogin();
  }

  async loginWithInvalidUser() {
    await this.enterCredentials('wrong_user', 'wrong_pass');
    await this.clickLogin();
  }
}

module.exports = LoginPage;
```

## 8. pages/InventoryPage.js
```javascript
const { expect } = require('@playwright/test');

class InventoryPage {
  constructor(page) {
    this.page = page;
    this.sortDropdown = '[data-test="product-sort-container"]';
    this.productList = '.inventory_item';
    this.productTitle = '.inventory_item_name';
    this.productCost = '.inventory_item_price';
    this.addToCartBtn = 'button.btn_inventory';
    this.cartIcon = '.shopping_cart_link';
    this.cartBadge = '.shopping_cart_badge';
  }

  async verifyPageLoaded() {
    await expect(this.page.locator(this.sortDropdown)).toBeVisible();
  }

  async changeSortOrder(sortType) {
    await this.page.selectOption(this.sortDropdown, sortType);
  }

  async getProductDetails() {
    const products = [];
    const items = this.page.locator(this.productList);
    const itemCount = await items.count();

    for (let i = 0; i < itemCount; i++) {
      const product = {
        name: await items.nth(i).locator(this.productTitle).textContent(),
        price: await items.nth(i).locator(this.productCost).textContent(),
        itemElement: items.nth(i)
      };
      products.push(product);
    }
    return products;
  }

  async findLowestPricedItems(quantity) {
    const allProducts = await this.getProductDetails();
    const sortedByPrice = allProducts.sort((item1, item2) => {
      const price1 = parseFloat(item1.price.replace('$', ''));
      const price2 = parseFloat(item2.price.replace('$', ''));
      return price1 - price2;
    });
    return sortedByPrice.slice(0, quantity);
  }

  async addLowCostItemsToCart(quantity) {
    const lowPriceItems = await this.findLowestPricedItems(quantity);
    const selectedItems = [];

    for (const item of lowPriceItems) {
      await item.itemElement.locator(this.addToCartBtn).click();
      selectedItems.push({
        name: item.name,
        price: item.price
      });
    }
    return selectedItems;
  }

  async getCartQuantity() {
    const badge = this.page.locator(this.cartBadge);
    if (await badge.isVisible()) {
      return parseInt(await badge.textContent());
    }
    return 0;
  }

  async openCart() {
    await this.page.click(this.cartIcon);
  }
}

module.exports = InventoryPage;
```

## 9. pages/CartPage.js
```javascript
const { expect } = require('@playwright/test');

class CartPage {
  constructor(page) {
    this.page = page;
    this.cartItem = '.cart_item';
    this.itemTitle = '.inventory_item_name';
    this.itemPrice = '.inventory_item_price';
    this.itemQty = '.cart_quantity';
    this.checkoutBtn = '[data-test="checkout"]';
  }

  async verifyPageLoaded() {
    await expect(this.page.locator(this.cartItem).first()).toBeVisible();
  }

  async getItemsInCart() {
    const cartItems = [];
    const items = this.page.locator(this.cartItem);
    const itemCount = await items.count();

    for (let i = 0; i < itemCount; i++) {
      const item = {
        name: await items.nth(i).locator(this.itemTitle).textContent(),
        price: await items.nth(i).locator(this.itemPrice).textContent(),
        quantity: await items.nth(i).locator(this.itemQty).textContent()
      };
      cartItems.push(item);
    }
    return cartItems;
  }

  async checkCartContents(expectedItems) {
    const actualCartItems = await this.getItemsInCart();
    
    expect(actualCartItems.length).toBe(expectedItems.length);
    
    for (let i = 0; i < expectedItems.length; i++) {
      expect(actualCartItems[i].name).toBe(expectedItems[i].name);
      expect(actualCartItems[i].price).toBe(expectedItems[i].price);
      expect(actualCartItems[i].quantity).toBe('1');
    }
  }

  async startCheckout() {
    await this.page.click(this.checkoutBtn);
  }

  async countTotalItems() {
    const items = await this.getItemsInCart();
    return items.reduce((total, item) => total + parseInt(item.quantity), 0);
  }
}

module.exports = CartPage;
```

## 10. pages/CheckoutPage.js
```javascript
const { expect } = require('@playwright/test');

class CheckoutPage {
  constructor(page) {
    this.page = page;
    this.firstNameField = '[data-test="firstName"]';
    this.lastNameField = '[data-test="lastName"]';
    this.zipCodeField = '[data-test="postalCode"]';
    this.continueBtn = '[data-test="continue"]';
  }

  async verifyPageLoaded() {
    await expect(this.page.locator(this.firstNameField)).toBeVisible();
  }

  async enterCustomerDetails(details) {
    await this.page.fill(this.firstNameField, details.firstName);
    await this.page.fill(this.lastNameField, details.lastName);
    await this.page.fill(this.zipCodeField, details.postalCode);
  }

  async goToOverview() {
    await this.page.click(this.continueBtn);
  }
}

module.exports = CheckoutPage;
```

## 11. pages/ConfirmationPage.js
```javascript
const { expect } = require('@playwright/test');

class ConfirmationPage {
  constructor(page) {
    this.page = page;
    this.finishBtn = '[data-test="finish"]';
    this.subtotalLabel = '.summary_subtotal_label';
    this.taxLabel = '.summary_tax_label';
    this.totalLabel = '.summary_total_label';
    this.successMsg = '.complete-header';
  }

  async verifyPageLoaded() {
    await expect(this.page.locator(this.finishBtn)).toBeVisible();
  }

  async getPriceSummary() {
    return {
      subtotal: await this.page.textContent(this.subtotalLabel),
      tax: await this.page.textContent(this.taxLabel),
      total: await this.page.textContent(this.totalLabel)
    };
  }

  async verifyPricing(items) {
    const summary = await this.getPriceSummary();
    
    const itemsTotal = items.reduce((sum, item) => {
      return sum + parseFloat(item.price.replace('$', ''));
    }, 0);
    
    const expectedSubtotal = `Item total: $${itemsTotal.toFixed(2)}`;
    expect(summary.subtotal).toBe(expectedSubtotal);
    
    const taxAmount = parseFloat(summary.tax.replace('Tax: $', ''));
    const finalTotal = parseFloat(summary.total.replace('Total: $', ''));
    
    const calculatedTotal = itemsTotal + taxAmount;
    expect(finalTotal).toBeCloseTo(calculatedTotal, 2);
    
    return summary;
  }

  async finalizeOrder() {
    await this.page.click(this.finishBtn);
  }

  async getConfirmationMessage() {
    await expect(this.page.locator(this.successMsg)).toBeVisible();
    return await this.page.textContent(this.successMsg);
  }
}

module.exports = ConfirmationPage;
```

## 12. data/test-data.json
```json
{
  "users": {
    "standard": {
      "username": "standard_user",
      "password": "secret_sauce"
    },
    "invalid": {
      "username": "invalid_user",
      "password": "wrong_password"
    }
  },
  "customer": {
    "firstName": "Test",
    "lastName": "User",
    "postalCode": "12345"
  },
  "errorMessages": {
    "invalidLogin": "Epic sadface: Username and password do not match any user in this service"
  }
}
```

## 13. utils/helpers.js
```javascript
class TestHelpers {
  static async executeWithRetry(action, maxAttempts, waitTime) {
    try {
      return await action();
    } catch (error) {
      if (maxAttempts === 0) throw error;
      await new Promise(resolve => setTimeout(resolve, waitTime));
      return this.executeWithRetry(action, maxAttempts - 1, waitTime);
    }
  }

  static convertPriceToNumber(priceString) {
    return parseFloat(priceString.replace('$', ''));
  }
}

module.exports = TestHelpers;
```

## 14. .github/workflows/playwright-tests.yml
```yaml
name: Run Playwright Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test-execution:
    timeout-minutes: 60
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    
    - uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install packages
      run: npm ci
      
    - name: Setup Playwright
      run: npx playwright install --with-deps
      
    - name: Check code style
      run: npm run lint
      
    - name: Execute tests
      run: npm run test:regression
      
    - uses: actions/upload-artifact@v3
      if: always()
      with:
        name: test-reports
        path: reports/html-report/
        retention-days: 30
```

## Setup Instructions:

1. **Create the project directory:**
```bash
mkdir saucedemo-automation
cd saucedemo-automation
```

2. **Create all the files with the content above**

3. **Initialize the project:**
```bash
npm init -y
```

4. **Install dependencies:**
```bash
npm install @playwright/test
npm install --save-dev eslint prettier
```

5. **Install Playwright browsers:**
```bash
npx playwright install
```

6. **Run the tests:**
```bash
npm test
```
