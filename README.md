# electron+vue3+ts+vite 从零开始搭建一个项目(1)

## 背景

**本人最近在学习`electron`桌面端开发，作为一个前端小白，在与`vue3`和`typescript`结合时踩了不少坑，网上现有的资料或视频大多讲诉不够清晰或者较为陈旧，容易使人知其然但不知其所以然，遂写下此文章以记录项目搭建过程，以供参考。废话不多说，我们开始。**

## 项目搭建过程

**首先使用`vite`工具创建一个`vue3+ts`的项目。命令如下：**

```
npm create vite
```

**框架我们选择`vue+ts`。之后我们运行以下命令来启动这个项目。**

```
cd test-project
npm i
npm run dev
```

**接下来访问`http://127.0.0.1:5173/`地址可以看到项目已经启动成功。**

![Alt text](../%E5%9B%BE%E7%89%87/image-20230510172801495.png)

**接下来我们安装`electron`，使用以下命令（注意，我们这里使用 `-D` 来安装开发依赖，之后所使用到的包均为开发依赖，不再过多赘述。：**

```
npm i -D electron
```

**同时，在根目录创建一个`electron`文件夹，并在里面新建两个文件，分别是`main.ts`和`preload.ts`。**

```javascript
// main.ts

// 控制应用生命周期和创建原生浏览器窗口的模组
const { app, BrowserWindow } = require("electron");
const path = require("path");

function createWindow() {
  // 创建浏览器窗口
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      // 引入预加载文件
      preload: path.join(__dirname, "preload.js"),
    },
  });
    
  // 加载vite启动的本地服务
  mainWindow.loadURL("http://localhost:5173");

}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
  createWindow();

  app.on("activate", function () {
    // 通常在 macOS 上，当点击 dock 中的应用程序图标时，如果没有其他
    // 打开的窗口，那么程序会重新创建一个窗口。
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
  });
});

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。 因此，通常对程序和它们在
// 任务栏上的图标来说，应当保持活跃状态，直到用户使用 Cmd + Q 退出。
app.on("window-all-closed", function () {
  if (process.platform !== "darwin") app.quit();
});
```

```javascript
// preload.ts

// 这里我们随便写一句话
console.log("preload");
```

**由于`main.ts`和`preload.ts`均为ts文件，需要编译为`js`文件，因此在`tsconfig.json`中进行配置：**

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": false, // 这里设为false
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": false, // 这里设为false
    "outDir": "dist" // 这里设置输出文件和vite打包后的文件一致，保证main.js和preload.js和打包后的index.html在同一路径
  },
  "include": ["electron/*.ts"], // 要编译的文件
  "references": [{ "path": "./tsconfig.node.json" }]
}

```

**之后再`package.json`中进行配置：**

```json
{
  "name": "test-project",
  "private": true,
  "version": "0.0.0",
  "main": "dist/main.js", //设置入口文件，即main.ts编译后的文件
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",  // 这里需要在vue-tsc后面加上 --noEmit
    "preview": "vite preview",
    "electron:dev":"tsc && electron ." // 这里是启动electron命令，先通过tsc编译ts文件，再运行electron .
  },
  ···
}
```



**由于我们在`main.ts`使用加载本地服务的方式来访问`vue`页面，因此我们需要先启动本地服务。**

```
npm run dev
```

**等待服务开启，运行以下命令即可打开electron窗口：**

```
npm run electron:dev
```

![Alt text](../%E5%9B%BE%E7%89%87/image-20230510201153201.png)

**至此，一个简单的`electron+vue3+ts+vite`应用就创建完成了。**

****

**看到这里可能有人会发现，我们每次都要先启动服务，等到服务开启之后才能执行`electron:dev`命令，能不能做到一个命令解决呢？答案肯定是可以的，下一节我将会介绍几个好用的工具来提升我们的开发效率。**



# electron+vue3+ts+vite 从零开始搭建一个项目(2)

**上一节我们讲到了从零搭建一个`electron+vue+ts+vite`的项目，这一节我将介绍几个好用的工具，来帮助我们更好的开发electron项目。**

------

**首先 ，我们在`main.ts`中加上这段代码：**

```typescript
// main.ts
···
// 判断当前环境是否为开发环境
if (process.env.NODE_ENV === "development") {
  // 当处于开发环境时，页面加载本地服务，并自动打开开发者工具
  mainWindow.loadURL("http://localhost:5173");
  mainWindow.webContents.openDevTools();
} else {
  // 否则页面加载打包后的index.html文件
  mainWindow.loadFile(path.join(__dirname, "./index.html"));
}
···
```

**这段代码中我们希望通过判断开发环境来加载不同的服务或文件，并判断是否要自动打开开发者工具，但是当我们重新启动项目时会发现页面为空白，也不能自动打开开发者工具，并且出现了以下错误：**

![Alt text](../%E5%9B%BE%E7%89%87/image-20230510212920208.png)

**很明显，`process.env.NODE_ENV`并不等于`development`，因此页面会去加载同一层级的`index.html`，这是因为我们在运行命令时并没有把`NODE_ENV`加进来，但由于跨平台，直接在命令前添加`NODE_ENV=development`会报错，因此我们需要使用`cross-env`这个工具来解决。**

**首先安装这个工具：**

```cmd
npm i -D cross-env
```

**然后，我们需要修改`package.json`文件中的`electron:dev`命令：**

```json
···
"scripts": {
   "dev": "vite",
   "build": "vue-tsc --noEmit && vite build",
   "preview": "vite preview",
   "electron:dev": "tsc && cross-env NODE_ENV=development electron ." // 在 electron . 前增加 cross-env NODE_ENV=development
},
···
```

**修改完之后，我们再次运行`npm run electron:dev`命令，发现页面可以正常加载，并且自动打开开发者工具。 **

![Alt text](../%E5%9B%BE%E7%89%87/image-20230510214217808.png)

------

**再启动项目时，每次都需要先开启本地`vite`服务，等待服务开启之后，才能进行启动`electron`并加载合适的页面，但这不免有些麻烦，这时有些小伙伴可能会提出，我们把这些命令写在一行，用`&&`符号连接起来不就好了，这看似很合理，但其实不行，这是因为当我们运行完 `npm run dev`时，后面的命令并不会继续执行，不信的小伙伴可以尝试一下。**

**为了解决这个问题，我们需要`concurrently`和 `wait-on`这两个工具，其中：**

- **`concurrently`可以帮助我们同时运行多个命令**
- **`wait-on`可以帮助我们等待某个命令执行完之后再去执行后面的命令**

**因此我们来安装这两个工具：**

```cmd
npm i -D concurrently wait-on 
```

**同时，我们需要修改`package.json`文件：**

```json
···
"scripts": {
   "dev": "vite",
   "build": "vue-tsc --noEmit && vite build",
   "preview": "vite preview",
   "electron":"wait-on tcp:5173 && tsc && cross-env NODE_ENV=development electron .", // 等待5173端口服务开启，再去运行之后的命令
   "electron:dev": "concurrently -k \"npm run dev \" \"npm run electron\"" // 同时运行多个命令
},
···
```

**修改好之后，再去运行`npm run electron:dev`命令，就可以一键启动`electron`项目了**

![Alt text](../%E5%9B%BE%E7%89%87/image-20230510220351695.png)

**至此结束。**

# electron+vue3+ts+vite 从零开始搭建一个项目(3)

**这一节我们来讲以下`electron`项目的打包。**

------

**`electron`打包主要用到的工具为`electron-builder`，首先安装该工具：**

```cmd
npm i -D electron-builder
```

**然后，在`package.json`文件中增加`build`配置项，并进行如下配置：**

```json
···
"build":{
  "appId": "my-app", // app的id
  "productName": "el-vue-test", // 项目名称
  "copyright": "Copyright © 2021 <condingandsleeping>", // 版权信息
  "mac": { // mac中的相关打包配置
      "category": "public.app-category.utilities"
  },
  "nsis": { // windows中安装程序配置
      "oneClick": false,
      "allowToChangeInstallationDirectory": true
  },
  "files": [ 
      "dist" // 要打包的文件夹
  ],
  "directories": {
      "buildResources": "assets",
      "output": "build" // 输出文件夹
  }
}
···
```

**除了以上配置之外，还有很多相关配置，具体可以查阅`electron-builder`的官方文档。**

**同时，在`package.json`文件中增加命令`electron:build`：**

```json
···
"scripts": {
  "dev": "vite",
  "build": "vue-tsc --noEmit && vite build",
  "preview": "vite preview",
  "electron": "wait-on tcp:5173 && tsc && cross-env NODE_ENV=development electron .",
  "electron:dev": "concurrently -k \"npm run dev \" \"npm run electron\""，
  "electron:build":"npm run build && tsc && electron-builder" // 打包命令，执行顺序为 vite打包、tsc编译、electron打包
},
···
```

**最后，修改vite.config.ts:**

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  base:"./", // 增加相对路径配置项，保证index.html可以找到打包后的js和css文件
  plugins: [vue()],
})
```

**执行`npm run electron:build`进行打包，打包后会增加两个文件夹，具体的文件目录为：**

```
···
dist
 -assets
 -index.html
 -main.js
 -preload.js
 -vite.svg
 
build
 -win-unpacked
 -builder-debug.yml
 -builder-effective-config.yaml
 -test-project Setup 0.0.0.exe
 -test-project Setup 0.0.0.exe.blockmap
···
```

**至此，一个完整的electron项目就打包完成了**

------

**完整的配置文件如下：**

```json
// package.json
{
  "name": "test-project",
  "private": true,
  "version": "0.0.0",
  "main": "dist/main.js",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "preview": "vite preview",
    "electron": "wait-on tcp:5173 && tsc && cross-env NODE_ENV=development electron .",
    "electron:dev": "concurrently -k \"npm run dev \" \"npm run electron\"",
    "electron:build":"npm run build && tsc && electron-builder"
  },
  "dependencies": {
    "vue": "^3.2.47"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.1.0",
    "concurrently": "^8.0.1",
    "cross-env": "^7.0.3",
    "electron": "^24.2.0",
    "electron-builder": "^23.6.0",
    "typescript": "^4.9.3",
    "vite": "^4.2.0",
    "vue-tsc": "^1.2.0",
    "wait-on": "^7.0.1"
  },
  "build":{
    "appId": "my-app",
    "productName": "test-project",
    "copyright": "Copyright © 2021 <xxx>",
    "mac": {
      "category": "public.app-category.utilities"
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true
    },
    "files": [
      "dist"
    ],
    "directories": {
      "buildResources": "assets",
      "output": "build"
    }
  }
}

```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": false,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": false,
    "outDir": "dist"
  },
  "include": ["electron/*.ts"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  base:"./",
  plugins: [vue()],
})
```

