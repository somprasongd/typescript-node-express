# Using TypeScript with Node.js and Express

วิธีการตั้งค่าโปรเจคให้สามารถรัน TypeScript ได้ พร้อมกับวิธีการนำไปใช้กับ Express

## Prerequisites

- Node.js version >= 12
- npm หรือ yarn

## การตั้งค่าโปรเจคให้สามารถรัน TypeScript ได้

- เริ่มจากสร้างโปรเจคใหม่ และสร้างไฟล์ package.json

```bash
mkdir typescript-node
cd typescript-node
npm init -y
```

- ติดตั้ง libraries ใน devDepencies

  - `typescript` เป็นตัวหลักที่ช่วยแปลงจาก TypeScript ไปเป็น JavaScript
  - `ts-node` เป็นเหมือน dev server ค่อยช่วยแปลงโค้ดจาก TypeScript ไปเป็น JavaScript ในขั้นตอนพัฒนา
  - `nodemon` ตัวช่วยให้รันโค้ดใหม่อัตโนมัติ เมื่อมีการแก้ไขโค้ด ทำงานร่วมกับ `ts-node`

```bash
npm i -D typescript ts-node nodemon
```

- สร้างไฟล์ tsconfig.json เพื่อกำหนดการตั้งค่าต่างๆ

```json
{
  "compilerOptions": {
    "target": "es6" /* กำหนดว่าให้แปลงเป็น js version อะไร */,
    "module": "commonjs" /* กำหนดว่าจะใช้ module แบบไหน */,
    "moduleResolution": "node" /* บอกใช้กับ Node.js */,
    "esModuleInterop": true /* อนุญาติให้ complie ES6 module เป็น commonjs*/,
    "strict": true /* เปิดใช้ strict type-checking options*/,
    "outDir": "./dist" /* กำหนดชื่อโฟลเดอร์ output (JavaScript) */,
    "rootDir": "./src" /* กำหนดชื่อโฟลเดอร์ sourcecode (TypeScript) */,
    "jsx": "react" /* แถมกรณี React จะมีไฟล์ .tsx ให้คอมไพล์เป็น .jsx ด้วย */
  }
}
```

- แก้ไข script ไฟล์ package.json

  - `dev` สำหรับรันโค้ดในโหมดพัฒนา จะทำการ reload อัตโนมัติเมื่อแก้ไขโค้ด
  - `build` สำหรับแปลงไฟล์เป็น JavaScript สำหรับการนำไปใช้งานจริง

```json
"scripts": {
  "dev": "nodemon",
  "build": "tsc --project ./"
},
```

- และสร้างไฟล์ nodemon.json

```json
{
  "watch": ["src"],
  "ext": "ts",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node ./src/index.ts"
}
```

## ทดสอบโค้ด TypeScript

- สร้างไฟล์ src/index.ts

```ts
const geeting = (name: string) => {
  console.log(`Hello ${name} from TypeScript.`);
};

geeting('Ball');
```

- รัน `npm run dev`

```bash
> typescript-node@1.0.0 dev D:\workspaces\playgounds\typescript-node
> nodemon

[nodemon] 2.0.7
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src\**\*
[nodemon] watching extensions: ts
[nodemon] starting `ts-node ./src/index.ts`
Hello Ball from TypeScript.
[nodemon] clean exit - waiting for changes before restart
```

- ทดลองแก้ไขชื่อ `nodemon` ก็จะ restart ให้อัตโนมัติ

```bash
[nodemon] restarting due to changes...
[nodemon] starting `ts-node ./src/index.ts`
Hello Stamp from TypeScript.
[nodemon] clean exit - waiting for changes before restart
```

- ทดสอบ build โดยรันคำสั่ง `npm run build` ก็ได้จะได้โฟลเดอร์ `dist` ออกมา และมีไฟล์ index.js อยู่ข้างใน

```js
// dist/index.js
'use strict';
const geeting = (name) => {
  console.log(`Hello ${name} from TypeScript.`);
};
geeting('Stamp');
```

## วิธีตั้งค่า TypeScript กับ Express

- ติดตั้ง Express รัน `npm i express`

- สร้าง Express Server ง่ายๆ ที่ไฟล์ src/index.ts

```ts
import express from 'express';

const app = express();

const PORT = 8000;

app.get('/', (req, res) => res.send('Express + TypeScript Server'));

app.listen(PORT, () => {
  console.log(`Server is running at http://localhost:${PORT}`);
});
```

- ทดสอบรัน npm run dev จะพบว่าไม่สามารถรันได้ เนื่องจาก มันไม่รู้จัก type นั่นเอง

- แก้ไขโดยการติดตั้ง @types ที่ต้องการเพิ่มเข้าไป

```bash
npm i -D @types/node @types/express
```

> ถ้าใช้ bodyParser ก็ต้องติดตั้ง @types/body-parser เพิ่มด้วย

- เมื่อรองรันใหม่อีกครั้งก็จะสามารถรันได้แล้ว

```bash
npm run dev

> typescript-node@1.0.0 dev D:\workspaces\playgounds\typescript-node
> nodemon

[nodemon] 2.0.7
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): src\**\*
[nodemon] watching extensions: ts
[nodemon] starting `ts-node ./src/index.ts`
Server is running at http://localhost:8000
```

### ยังๆ... ไม่หมด

เนื่องจากเราเขียนแบบ TypeScript แต่โค้ดข้างต้นยังไม่ได้ถูกกำหนด type ให้ถูกต้องเลย ดังนั้นเรามากำหนด type ให้ถูกต้องการ ดังนี้

- app ต้องมี type เป็น Application
- req ต้องมี type เป็น Request
- res ต้องมี type เป็น Response

```ts
import express, { Application, Request, Response } from 'express';

const app: Application = express();

const PORT: number = 8000;

app.get('/', (req: Request, res: Response) => {
  res.send('Express + TypeScript Server');
});

app.listen(PORT, () => {
  console.log(`Server is running at http://localhost:${PORT}`);
});
```

เท่านี้ก็เรียบร้อยแล้ว
