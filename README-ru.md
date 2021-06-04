# Invoice.SDK.Node.js

В репозитории находятся SDK для интеграции и простой пример для интеграции Invoice, используя Node.js

## Примеры

[Более подробная информация по работе API](https://dev.invoice.su)

Создание терминала

```javascript
const setTerminalInfo = async (shop, restClient) => {
  try {
    const terminal = new GetTerminal(shop.id);
    const terminalInfo = await restClient.getTerminal(terminal);

    if (terminalInfo.data.id) {
      return terminalInfo.data;
    }

    const newTerminal = new CreateTerminal(shop.name, shop.id);
    newTerminal.setDescription('Онлайн оплата');
    const newTerminalInfo = await restClient.createTerminal(newTerminal);

    return newTerminalInfo;
  } catch (err) {
    console.log('Произошла ошибка \n', err);
    return Promise.reject(err);
  }
};
