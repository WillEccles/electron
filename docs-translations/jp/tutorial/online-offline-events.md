# オンライン/オフライン イベントの検知

オンラインとオフラインイベントの検知は、標準のHTML 5 APIを使用して、状況表示を実装できます。次の例を見てみましょう。

_main.js_

```javascript
const electron = require('electron');
const app = electron.app;
const BrowserWindow = electron.BrowserWindow;

var onlineStatusWindow;
app.on('ready', function() {
  onlineStatusWindow = new BrowserWindow({ width: 0, height: 0, show: false });
  onlineStatusWindow.loadURL('file://' + __dirname + '/online-status.html');
});
```

_online-status.html_

```html
<!DOCTYPE html>
<html>
<body>
<script>
  var alertOnlineStatus = function() {
    window.alert(navigator.onLine ? 'online' : 'offline');
  };

  window.addEventListener('online',  alertOnlineStatus);
  window.addEventListener('offline',  alertOnlineStatus);

  alertOnlineStatus();
</script>
</body>
</html>
```

メインプロセスでそれらのイベントに反応するインスタンスがあるかもしれません。メインプロセスは、`navigator` オブジェクトをもたず、直接それらのイベントを検知できません。Electronの内部プロセスコミュニケーションツールを使用して、イベントはメインプロセスに転送され、必要があればハンドルできます。次の例を見てみましょう。

_main.js_

```javascript
const electron = require('electron');
const app = electron.app;
const ipcMain = electron.ipcMain;
const BrowserWindow = electron.BrowserWindow;

var onlineStatusWindow;
app.on('ready', function() {
  onlineStatusWindow = new BrowserWindow({ width: 0, height: 0, show: false });
  onlineStatusWindow.loadURL('file://' + __dirname + '/online-status.html');
});

ipcMain.on('online-status-changed', function(event, status) {
  console.log(status);
});
```

_online-status.html_

```html
<!DOCTYPE html>
<html>
<body>
<script>
  const ipcRenderer = require('electron').ipcRenderer;
  var updateOnlineStatus = function() {
    ipcRenderer.send('online-status-changed', navigator.onLine ? 'online' : 'offline');
  };

  window.addEventListener('online',  updateOnlineStatus);
  window.addEventListener('offline',  updateOnlineStatus);

  updateOnlineStatus();
</script>
</body>
</html>
```
