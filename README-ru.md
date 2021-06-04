# Invoice.SDK.Node.js

В репозитории находятся SDK для интеграции и простой пример для интеграции Invoice, используя Node.js

## Примеры

[Более подробная информация по работе API](https://dev.invoice.su)

### Создание терминала

```javascript
const RestClient = require('sdk/rest-client');
const CreateTerminal = require('sdk/templates/create-terminal');
const GetTerminal = require('sdk/templates/get-terminal');

const login = "demo"; //Логин от lk.invoice.su
const api_key = "1526fec01b5d11f4df4f2160627ce351"; // API ключ

const shop = {
  id: '28b839d026189922', // Store id in the system
  name: 'Магазин игрушек', // Store name in the system
};

/*
 * Create a rest client instance
 *
 * login - User login
 * apiKey - API key, which can be obtained / changed in the "Settings" section
 */

const restClient = new RestClient(login, apiKey);

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
    newTerminal.setDescription('Онлайн оплата');
    const newTerminalInfo = await restClient.createTerminal(newTerminal);

    return newTerminalInfo;
  } catch (err) {
    return Promise.reject(err);
  }
};
```
