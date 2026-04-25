# Greed Snake - 代码说明

## 文件结构

```
Greed_Snake/
├── index.html          ← 全部代码（HTML + CSS + JS，单文件无依赖）
├── RUN_GUIDE.md        ← 运行指南
└── SPEC.md             ← 游戏设计规格文档
```

所有逻辑全部在 `index.html` 中，分为以下几个代码区域（行号对应文件内位置）：

| 区域 | 行号 | 说明 |
|------|------|------|
| CONSTANTS | 92–112 | 游戏全局配置参数 |
| AUDIO | 114–183 | Web Audio API 音效合成 |
| STATE | 185–219 | 游戏状态数据结构 |
| CANVAS | 221–224 | Canvas DOM 引用 |
| INPUT | 226–281 | 键盘事件处理 |
| HELPERS | 283–307 | 工具函数 |
| GAME LOGIC | 309–551 | 核心游戏逻辑 |
| RENDERING | 553–668 | Canvas 绘制 |
| GAME LOOP | 670–688 | requestAnimationFrame 主循环 |
| INIT | 690–691 | 启动入口 |

---

## 常量配置（92–112行）

```js
const GRID_SIZE = 20;       // 网格：20×20 格子
const CELL_SIZE = 32;       // 每格 32px，canvas 总尺寸 640×640
const BASE_SPEED = 120;     // 基础移动间隔 120ms/步
const LEVEL_THRESHOLDS = [50, 120, 220, 350, 500];  // 各关卡分数阈值
const OBSTACLE_COUNT = 10;   // 每关障碍物数量
const POWERUP_COUNT = 3;    // 每关道具数量
const POWERUP_DURATION = 5000;  // 道具效果持续 5000ms
```

调整这些常量即可修改游戏难度和平衡性。颜色方案定义在 `COLORS` 对象中。

---

## 状态管理（202–219行）

游戏状态由一个顶层 `state` 对象管理：

```js
let state = {
  gameState: GameState.PLAYING,  // START | PLAYING | PAUSED | LEVEL_UP | GAME_OVER | WIN
  snake: [],                      // 蛇身数组，索引0为蛇头
  direction: Direction.RIGHT,     // 当前移动方向
  nextDirection: Direction.RIGHT, // 下一帧方向（缓冲，防止快速按键错乱）
  food: null,                     // 食物坐标
  obstacles: [],                  // 障碍物数组（含 moving, moveDir 属性）
  powerUps: [],                   // 道具数组（含 type: 'speedUp'|'speedDown'）
  score: 0,
  level: 1,
  highScore: parseInt(localStorage.getItem('greedSnakeHighScore')) || 0,
  speed: BASE_SPEED,
  speedMultiplier: 1,             // 速度倍率，1.5=加速，0.75=减速
  speedTimer: null,              // 道具效果定时器
  lastMoveTime: 0,              // 上次移动时间戳（用于固定步长）
  levelUpTimer: 0,
  pulsePhase: 0                  // 食物脉冲动画相位
};
```

---

## 碰撞检测（394–465行）

`moveSnake()` 逐帧执行，顺序如下：

```
1. 移动到新位置（nextDirection）
2. 边界检测 → 穿墙到对侧（无限地图）
3. 自撞检测 → 从碰撞点截断蛇身，<1节则 gameOver()
4. 障碍物检测 → 缩短蛇身1节，<1节则 gameOver()
5. 蛇头放入数组（unshift）
6. 食物检测 → 加分、增长、检查关卡
7. 道具检测 → 应用速度倍率，启动5秒定时器
```

关键细节：
- 步骤2穿墙逻辑在 `unshift` 之前，保证新蛇头位置正确
- 自撞检测使用 `findIndex` 而非 `some`，可以定位碰撞位置，用 `slice` 截断而非只缩短尾部 — 这样撞到身体中间时不会把头尾都留下
- 障碍物碰撞和自撞的死亡条件统一为 `< 1` 节

---

## 关卡系统（484–497行）

`checkLevelUp()` 在每次吃食物后调用：

```js
if (state.level < 5 && state.score >= LEVEL_THRESHOLDS[state.level - 1]) {
  state.level++;
  spawnObstacles();   // 重新生成障碍物
  spawnPowerUps();    // 重新生成道具
  state.speed = Math.max(60, BASE_SPEED - (state.level - 1) * 10);  // 加速
} else if (state.level === 5 && state.score >= LEVEL_THRESHOLDS[4]) {
  win();  // 通关
}
```

关卡门槛分数存在 `LEVEL_THRESHOLDS` 数组中，索引从 0 开始。

---

## 音效系统（114–183行）

使用 Web Audio API 的 `OscillatorNode`（振荡器）+ `GainNode`（增益）合成所有声音，无需外部音频文件：

| 函数 | 触发时机 | 频率特征 |
|------|---------|---------|
| `playEat()` | 吃食物 | 正弦波 400→800Hz 滑音，150ms |
| `playPowerUp()` | 吃道具 | 660Hz + 880Hz 双音 |
| `playObstacleHit()` | 撞障碍物 | 100Hz 方波，80ms |
| `playGameOver()` | 撞墙/撞自己 | 400→300→200→100Hz 递减琶音 |
| `playLevelUp()` | 过关 | 523→659→784→1047Hz 递增琶音 |
| `playWallHit()` | 撞墙 | 80Hz 锯齿波 |
| `playWin()` | 通关 | C4→E4→G4→C5→G4→E4→C4 胜利音阶 |

`ensureAudio()` 在每次按键时调用，延迟创建 `AudioContext`，规避浏览器自动播放策略限制。

---

## 游戏循环（670–688行）

```js
function gameLoop(timestamp) {
  if (state.gameState === GameState.PLAYING) {
    moveObstacles(timestamp);   // 移动障碍物（200ms步长）
    const currentSpeed = state.speed / state.speedMultiplier;
    if (timestamp - state.lastMoveTime > currentSpeed) {
      moveSnake();
      state.lastMoveTime = timestamp;
    }
  }
  render(timestamp);            // 每帧都渲染
  requestAnimationFrame(gameLoop);
}
```

蛇和障碍物使用不同的时间步长：蛇的移动间隔由 `BASE_SPEED / speedMultiplier` 决定，障碍物固定 200ms 移动一次。两者都基于 `timestamp` 计算，保证在不同刷新率下行为一致。

---

## 渲染策略（553–668行）

每帧重绘整个 Canvas：
- 背景 + 网格线（固定不变，可考虑只画一次到离屏 canvas）
- 障碍物（灰色圆角矩形 + 发光）
- 食物（粉红色方块，带脉冲缩放动画）
- 道具（金色/蓝色圆形，带 +/- 文字图标）
- 蛇身（头部高亮 + 尾部渐变色彩）
- UI（分数、最高分、关卡、速度状态）

`pulsePhase` 累加用于食物动画：`scale = 0.9 + 0.2 * Math.sin(pulsePhase)`，产生平滑的呼吸效果。
