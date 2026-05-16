# 创建项目
```
npm create vite@latest 名字 -- --template react-ts
cd 名字
npm install
npm run dev
```
重点看：
```
src/App.tsx       主页面
src/main.tsx      入口文件
src/index.css     全局样式
package.json      项目依赖
tsconfig.json     TypeScript 配置
```
# TypeScript 基础类型
## TS 类型
```ts
const isOnline: boolean = true; 
```
`:`后为变量类型，此处的`boolean`为布尔类型
## type 定义结构类型
```ts
type Device = {
  id: number;
  name: string;
  status: string;
  temperature: number;
};
```
`type`类似于结构体，但是其只能用于开发阶段检查，并不是真的运行时结构体
还可以进行联合定义：如果超出定义范围`TS`会自动报错
```ts
type DeviceStatus = "online" | "offline";
//后续使用：
status:DeviceStatus
```
## 渲染设备数组
这段代码是在`App.tsx`中的
```ts
type DeviceStatus = "online" | "offline";
type Device = {  
	id: number;  
	name: string;  
	status: DeviceStatus;  
	temperature: number;  
};
const devices: Device[] = [  
	{  
		id: 1,  
		name: "ESP32 客厅节点",  
		status: "online",  
		temperature: 26.5  
	},  
	{  
		id: 2,  
		name: "ESP32 实验室节点",  
		status: "offline",  
		temperature: 0  
	},  
	{  
		id: 3,  
		name: "Jetson 边缘计算节点",  
		status: "online",  
		temperature: 48.2  
	}  
	];
function App() {  
	return (  
		<div>  
			<h1>设备管理面板</h1>  
		  
			{devices.map((device) => (  
				<div key={device.id}>  
					<h2>{device.name}</h2>  
					<p>设备 ID：{device.id}</p>  
					<p>状态：{device.status}</p>  
					<p>温度：{device.temperature} ℃</p>  
			</div>  
		))}  
		</div>  
	);  
}  
  
export default App;
```
`device`是一个数组，该数组中每个元素必须符合`Device`的类型定义
而这段代码的后半段是`React`中的，其重点是将`devices`遍历，并将`device`渲染成`div`
同时，如需要加单位(如上面的℃)，则在`React`代码中加
## AI辅助
示例`prompts`
```
我正在写 React + TypeScript 前端页面。

请基于下面的数据类型，帮我写一个设备卡片组件：

type DeviceStatus = "online" | "offline";

type Device = {
  id: number;
  name: string;
  status: DeviceStatus;
  temperature: number;
  battery: number;
};

要求：
1. 使用函数组件
2. props 必须有 TypeScript 类型
3. 在线状态显示绿色标签，离线状态显示灰色标签
4. 使用普通 CSS，不要用 UI 库
5. 代码要拆成 DeviceCard.tsx 和 DeviceCard.css
```
# 组件化开发
将之前写在`APP.tsx`中的代码拆成
```
src
├─ App.tsx            //负责页面整体结构和设备列表
├─ App.css
├─ components
│  ├─ DeviceCard.tsx  //负责单个设备卡片
│  └─ DeviceCard.css
└─ types
   └─ device.ts       //负责设备相关类型
```
## 文件：src/types/device.ts
```ts
export type DeviceStatus = "online" | "offline";
export type Device = {
  id: number;
  name: string;
  status: DeviceStatus;
  temperature: number;
  battery: number;
};
```
其中的`export`表示：这个类型可被别的文件导入
## 文件：src/components/DeviceCard.tsx
```ts
import type { Device, DeviceStatus } from "../types/device";
import "./DeviceCard.css";

type DeviceCardProps = {
  device: Device;
};

function getStatusText(status: DeviceStatus): string {
  if (status === "online") {
    return "在线";
  }

  return "离线";
}

function DeviceCard({ device }: DeviceCardProps) {
  const isLowBattery = device.battery < 20;

  return (
    <article className="device-card">
      <div className="device-header">
        <h2>{device.name}</h2>

        <span className={`status ${device.status}`}>
          {getStatusText(device.status)}
        </span>
      </div>

      <p>设备 ID：{device.id}</p>
      <p>温度：{device.temperature} ℃</p>
      <p>
        电量：{device.battery}%
        {isLowBattery && <span className="battery-warning"> 电量过低</span>}
      </p>
    </article>
  );
}

export default DeviceCard;
```
使用`import type`导入类型
组件函数`function DeviceCard({ device }: DeviceCardProps)`：
从该组件中接收`DeviceCardProps`→解构出`device`，类型为`Device`
为什么不直接写`Device`？
`React`实际传入的是一个`Props`对象，相当于将`Device`封装起来了
## 文件：src/components/DeviceCard.css
```css
.device-card {
  border: 1px solid #ddd;
  border-radius: 12px;
  padding: 16px 20px;
  background: #fff;
}

.device-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.device-header h2 {
  margin: 0;
  font-size: 20px;
}

.status {
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 14px;
}

.status.online {
  background: #e8f8ee;
  color: #167a3a;
}

.status.offline {
  background: #f3f4f6;
  color: #6b7280;
}
```
## 文件：src/App.tsx
```ts
import "./App.css";
import DeviceCard from "./components/DeviceCard";
import type { Device } from "./types/device";

const devices: Device[] = [
  {
    id: 1,
    name: "ESP32 客厅节点",
    status: "online",
    temperature: 26.5,
    battery: 85
  },
  {
    id: 2,
    name: "ESP32 实验室节点",
    status: "offline",
    temperature: 0,
    battery: 0
  },
  {
    id: 3,
    name: "Jetson 边缘计算节点",
    status: "online",
    temperature: 48.2,
    battery: 92
  }
];

function App() {
  return (
    <main className="page">
      <h1>设备管理面板</h1>

      <section className="device-list">
        {devices.map((device) => (
          <DeviceCard key={device.id} device={device} />
        ))}
      </section>
    </main>
  );
}

export default App;
```
其中，`<DeviceCard key={device.id} device={device} />`将当前的`device`作为`props`传给`DeviceCard`组件
而`props`就是父组件传给子组件的数据
## 文件：src/APP.css
```ts
.page {
  max-width: 900px;
  margin: 0 auto;
  padding: 32px;
  font-family: system-ui, sans-serif;
}

.device-list {
  display: grid;
  gap: 16px;
}
```
## AI辅助
```
我正在写 React + TypeScript 项目。

已有类型：

export type DeviceStatus = "online" | "offline";

export type Device = {
  id: number;
  name: string;
  status: DeviceStatus;
  temperature: number;
  battery: number;
  location: string;
};

请帮我写一个 DeviceCard 组件：
1. props 类型为 { device: Device }
2. 在线显示“在线”，离线显示“离线”
3. 电量低于 20 显示红色提醒
4. 样式拆到 DeviceCard.css
5. 不要使用 UI 库
```

# `useState` + 表单输入
## 在需要进行用户交互的过程中，需要将数据以“状态”形式传输
```ts
const [devices, setDevices] = useState<Device[]>(initialDevices);
```
- devices：当前设备数组
- setDevices：修改设备数组的函数
- useState<Device[]>：告诉 `TS`，这个状态是 `Device` 类型的数组
- initialDevices：初始数据
## 创建新数组，把新旧设备放进去
```ts
setDevices([...devices, newDevice]);
```
## 受控组件
```ts
			<input
              value={newName}
              onChange={(event) => setNewName(event.target.value)}
              placeholder="例如：ESP32 卧室节点"
            />
```
数据框显示内容由`newName`决定，用户输入内容时，通过`setNewName`修改`newName`，`newName`改变后输入框重新显示新内容
## 表单提交事件
```ts
<form className="device-form" onSubmit={handleAddDevice}>
```
表单提交时执行`handleAddDevice`
```ts
function handleAddDevice(event: FormEvent<HTMLFormElement>)
```
`event`是一个表单提交事件，该事件来自`HTMLFormElement`，在其后的`event.preventDefault();`阻止浏览器刷新页面
## 新增设备
```ts
const nextDevice: Device = {
  id: Date.now(),//由后端提供，此处为例方便
  name,
  location,
  status: newStatus,
  temperature,
  battery
};

setDevices([...devices, nextDevice]);
```
## 完整代码
### src/App.tsx
```ts
import { useState, type FormEvent } from "react";
import "./App.css";
import DeviceCard from "./components/DeviceCard";
import type { Device, DeviceStatus } from "./types/device";

const initialDevices: Device[] = [
  {
    id: 1,
    name: "ESP32 客厅节点",
    status: "online",
    temperature: 26.5,
    battery: 85,
    location: "客厅"
  },
  {
    id: 2,
    name: "ESP32 实验室节点",
    status: "offline",
    temperature: 0,
    battery: 0,
    location: "实验室"
  },
  {
    id: 3,
    name: "Jetson 边缘计算节点",
    status: "online",
    temperature: 48.2,
    battery: 92,
    location: "工作台"
  }
];

function App() {
  const [devices, setDevices] = useState<Device[]>(initialDevices);

  const [newName, setNewName] = useState("");
  const [newLocation, setNewLocation] = useState("");
  const [newStatus, setNewStatus] = useState<DeviceStatus>("online");
  const [temperatureInput, setTemperatureInput] = useState("25");
  const [batteryInput, setBatteryInput] = useState("80");

  function handleAddDevice(event: FormEvent<HTMLFormElement>) {
    event.preventDefault();

    const name = newName.trim();
    const location = newLocation.trim();
    const temperature = Number(temperatureInput);
    const battery = Number(batteryInput);

    if (!name || !location) {
      alert("请填写设备名称和位置");
      return;
    }

    if (Number.isNaN(temperature) || Number.isNaN(battery)) {
      alert("温度和电量必须是数字");
      return;
    }

    const nextDevice: Device = {
      id: Date.now(),
      name,
      location,
      status: newStatus,
      temperature,
      battery
    };

    setDevices([...devices, nextDevice]);

    setNewName("");
    setNewLocation("");
    setNewStatus("online");
    setTemperatureInput("25");
    setBatteryInput("80");
  }

  return (
    <main className="page">
      <h1>设备管理面板</h1>

      <form className="device-form" onSubmit={handleAddDevice}>
        <h2>新增设备</h2>

        <div className="form-grid">
          <label>
            设备名称
            <input
              value={newName}
              onChange={(event) => setNewName(event.target.value)}
              placeholder="例如：ESP32 卧室节点"
            />
          </label>

          <label>
            位置
            <input
              value={newLocation}
              onChange={(event) => setNewLocation(event.target.value)}
              placeholder="例如：卧室"
            />
          </label>

          <label>
            状态
            <select
              value={newStatus}
              onChange={(event) =>
                setNewStatus(event.target.value as DeviceStatus)
              }
            >
              <option value="online">在线</option>
              <option value="offline">离线</option>
            </select>
          </label>

          <label>
            温度
            <input
              type="number"
              value={temperatureInput}
              onChange={(event) => setTemperatureInput(event.target.value)}
            />
          </label>

          <label>
            电量
            <input
              type="number"
              value={batteryInput}
              onChange={(event) => setBatteryInput(event.target.value)}
            />
          </label>
        </div>

        <button type="submit">添加设备</button>
      </form>

      <section className="device-list">
        {devices.map((device) => (
          <DeviceCard key={device.id} device={device} />
        ))}
      </section>
    </main>
  );
}

export default App;
```
### src/App.css
```css
.page {
  max-width: 900px;
  margin: 0 auto;
  padding: 32px;
  font-family: system-ui, sans-serif;
}

.device-form {
  margin-bottom: 24px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 12px;
  background: #fafafa;
}

.device-form h2 {
  margin-top: 0;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 16px;
  margin-bottom: 16px;
}

.form-grid label {
  display: grid;
  gap: 6px;
  font-size: 14px;
}

.form-grid input,
.form-grid select {
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 14px;
}

.device-form button {
  padding: 8px 16px;
  border: none;
  border-radius: 8px;
  background: #111827;
  color: white;
  cursor: pointer;
}

.device-form button:hover {
  background: #374151;
}

.device-list {
  display: grid;
  gap: 16px;
}
```
## AI辅助
```
我正在写 React + TypeScript 项目。

已有类型：

export type DeviceStatus = "online" | "offline";

export type Device = {
  id: number;
  name: string;
  status: DeviceStatus;
  temperature: number;
  battery: number;
  location: string;
  remark?: string;
};

请帮我写一个新增设备表单组件：
1. 使用 useState 管理输入框
2. 提交后生成 Device 对象
3. props 需要包含 onAddDevice: (device: Device) => void
4. 表单包含 name、location、status、temperature、battery、remark
5. 使用 React + TypeScript
6. 使用普通 CSS
```