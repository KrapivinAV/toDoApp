# Invoice.SDK.Node.js

В репозитории находятся SDK для интеграции и простой пример для интеграции Invoice, используя Node.js

## Примеры

[Более подробная информация по работе API](https://dev.invoice.su)

### Создание терминала

```javascript
const RestClient = require('sdk/rest-client');
const CreateTerminal = require('sdk/templates/create-terminal');
const GetTerminal = require('sdk/templates/get-terminal');

const login = 'demo'; // User login
const api_key = '1526fec01b5d11f4df4f2160627ce351'; // API key, which can be obtained / changed in the "Settings" section

const shop = {
  id: '28b839d026189922', // Store id in the system
  name: 'Магазин игрушек', // Store name in the system
};

const restClient = new RestClient(login, apiKey); // Create a rest client instance

const setTerminalInfo = async (shop, restClient) => {
  try {
    // Check the existence of this terminal

    const terminal = new GetTerminal(shop.id);
    const terminalInfo = await restClient.getTerminal(terminal);

    if (terminalInfo.data.id) {
      return terminalInfo.data;
    }

    // Create terminal

    const newTerminal = new CreateTerminal(shop.name, shop.id);
    const newTerminalInfo = await restClient.createTerminal(newTerminal);

    return newTerminalInfo;
  } catch (err) {
    return Promise.reject(err);
  }
};
```

### Создание платежа

```javascript
const RestClient = require('../sdk/rest-client');
const CreatePayment = require('../sdk/templates/create-payment');
const Item = require('../sdk/templates/common/item');
const Order = require('../sdk/templates/common/order');
const Settings = require('../sdk/templates/common/settings');

const login = 'demo'; // User login
const api_key = '1526fec01b5d11f4df4f2160627ce351'; // API key, which can be obtained / changed in the "Settings" section
const terminalId = '9ad01d262144a13cda1e90593bf64479'; // Terminal ID in which the payment will be created
const orderId = 987987987; // Order ID
const items = [
  {
    name: 'Машинка', // Subject of the order
    price: 100, // Price for 1 unit
    count: 2, // Quantity
    fullPrice: 200, // Total price
  },
  {
    name: 'Робот', // Subject of the order
    price: 199, // Price for 1 unit
    count: 1, // Quantity
    fullPrice: 199, // Total price
  },
];
const customer = {
  name: 'Иван', // Customer name
  phone: '79991234567', // Customer phone number
  email: 'em@invoice.su', // Customer E-mail
};
const currency = 'RUB'; // Order currency
const successUrl = 'https://example.com/success.html'; // The link to which the user will be redirected in case of successful payment
const failUrl = 'https://example.com/fail.html'; // The link to which the user will be redirected in case of unsuccessful payment

const restClient = new RestClient(login, apiKey); // Create a rest client instance

const onPay = async () => {
  try {
    let amount = 0; // The total amount of the order
    let receipt = []; // Array with order items

    items.forEach((item) => {
      receipt.push(new Item(item.name, item.price, item.count, item.fullPrice));
      amount += item.fullPrice;
    });

    const order = new Order(amount, currency, 'Заказ №' + orderId, orderId);
    const settings = new Settings(terminalId, 'card', successUrl, failUrl);
    const createPayment = new CreatePayment(order, settings, receipt, customer);

    const paymentInfo = await restClient.createPayment(createPayment);

    return paymentInfo;
  } catch (err) {
    return Promise.reject(err);
  }
};

const paymentInfo = onPay();

if (paymentInfo.data.id) {
  console.log(
    'Платеж оформлен: https://pay.invoice.su/P' + paymentInfo.data.id,
  );
} else {
  console.log('Ошибка платежа ' + paymentInfo.data.description);
}
```

### Оформление возврата средств

```javascript
const RestClient = require('../sdk/rest-client');
const GetPaymentByOrder = require('../sdk/templates/get-payment-by-order');
const CreateRefund = require('../sdk/templates/create-refund');
const RefundInfo = require('../sdk/templates/common/refund-info');

const login = 'demo'; // User login
const api_key = '1526fec01b5d11f4df4f2160627ce351'; // API key, which can be obtained / changed in the "Settings" section
const paymentId = '126d4c806ef04b10f822541f1a5b41d9'; // Payment ID
const orderId = 987987987; // Order ID
const items = [
  {
    name: 'Машинка', // Subject of the order
    price: 100, // Price for 1 unit
    count: 2, // Quantity
    fullPrice: 200, // Total price
  },
  {
    name: 'Робот', // Subject of the order
    price: 199, // Price for 1 unit
    count: 1, // Quantity
    fullPrice: 199, // Total price
  },
];
const reason = 'Передумал'; // Reason for refund

const restClient = new RestClient(login, apiKey); // Create a rest client instance

const onRefund = async () => {
  try {
    const order = await restClient.getPaymentByOrder(
      new GetPaymentByOrder(orderId),
    );

    // Check the existence of this payment

    if (!order || order.error) {
      return false;
    }

    // Create refund

    let amount = 0; // The total amount of the order
    let receipt = []; // Array with order items

    items.forEach((item) => {
      receipt.push(new Item(item.name, item.price, item.count, item.fullPrice));
      amount += item.fullPrice;
    });

    const refundInfo = new RefundInfo(amount, reason, currency);
    const createRefund = new CreateRefund(order.data.id, refundInfo, receipt);

    const refundInfo = await restClient.createRefund(createRefund);

    return Boolean(refundInfo.data.status === 'successful');
  } catch (err) {
    return Promise.reject(err);
  }
};

const refundInfo = onPay();

if (refundInfo) {
  console.log('Возврат оформлен');
} else {
  console.log('Ошибка возврата ' + refundInfo.data.description);
}
```
