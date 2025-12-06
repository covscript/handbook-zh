# 3.4 ecs - 实体组件系统

ECS（Entity Component System）是一种用于游戏开发和模拟的架构模式。

### 基本概念

```covscript
import ecs

# 创建 ECS 世界
var world = ecs.createWorld()

# 定义组件
class Position
    var x = 0.0
    var y = 0.0
    
    function construct(px, py)
        this.x = px
        this.y = py
    end
end

class Velocity
    var dx = 0.0
    var dy = 0.0
    
    function construct(vx, vy)
        this.dx = vx
        this.dy = vy
    end
end

class Renderable
    var sprite = ""
    
    function construct(s)
        this.sprite = s
    end
end

# 创建实体
var entity1 = world.createEntity()
world.addComponent(entity1, new Position{100, 100})
world.addComponent(entity1, new Velocity{5, 0})
world.addComponent(entity1, new Renderable{"player.png"})

var entity2 = world.createEntity()
world.addComponent(entity2, new Position{200, 200})
world.addComponent(entity2, new Velocity{-3, 2})
world.addComponent(entity2, new Renderable{"enemy.png"})

# 定义系统
class MovementSystem
    function update(world, deltaTime)
        var entities = world.getEntitiesWith(Position, Velocity)
        
        foreach entity in entities
            var pos = world.getComponent(entity, Position)
            var vel = world.getComponent(entity, Velocity)
            
            pos.x += vel.dx * deltaTime
            pos.y += vel.dy * deltaTime
        end
    end
end

class RenderSystem
    function update(world, deltaTime)
        var entities = world.getEntitiesWith(Position, Renderable)
        
        foreach entity in entities
            var pos = world.getComponent(entity, Position)
            var ren = world.getComponent(entity, Renderable)
            
            # 渲染逻辑
            system.out.println("渲染 " + ren.sprite + " 于 (" +
                             to_string(pos.x) + ", " + to_string(pos.y) + ")")
        end
    end
end

# 注册系统
var movementSystem = new MovementSystem{}
var renderSystem = new RenderSystem{}

# 游戏循环
for frame = 0, frame < 10, ++frame
    var deltaTime = 0.016  # 约 60 FPS
    
    # 更新系统
    movementSystem.update(world, deltaTime)
    renderSystem.update(world, deltaTime)
    
    system.out.println("--- 帧 " + to_string(frame) + " ---")
    runtime.delay(16)
end
```

### 游戏示例：简单的射击游戏

```covscript
import ecs

# 组件定义
class Transform
    var x = 0.0
    var y = 0.0
    var rotation = 0.0
end

class Health
    var current = 100
    var max = 100
end

class Weapon
    var damage = 10
    var fireRate = 0.5
    var lastFire = 0.0
end

class Enemy
    var speed = 50.0
end

# 创建游戏世界
var gameWorld = ecs.createWorld()

# 创建玩家
function createPlayer(world, x, y)
    var player = world.createEntity()
    world.addComponent(player, new Transform{})
    world.getComponent(player, Transform).x = x
    world.getComponent(player, Transform).y = y
    world.addComponent(player, new Health{})
    world.addComponent(player, new Weapon{})
    return player
end

# 创建敌人
function createEnemy(world, x, y)
    var enemy = world.createEntity()
    world.addComponent(enemy, new Transform{})
    world.getComponent(enemy, Transform).x = x
    world.getComponent(enemy, Transform).y = y
    world.addComponent(enemy, new Health{})
    world.getComponent(enemy, Health).current = 50
    world.getComponent(enemy, Health).max = 50
    world.addComponent(enemy, new Enemy{})
    return enemy
end

# 系统：敌人移动
class EnemyMovementSystem
    function update(world, deltaTime)
        var enemies = world.getEntitiesWith(Transform, Enemy)
        
        foreach enemy in enemies
            var transform = world.getComponent(enemy, Transform)
            var enemyComp = world.getComponent(enemy, Enemy)
            
            # 向玩家移动（简化版，向左移动）
            transform.x -= enemyComp.speed * deltaTime
        end
    end
end

# 系统：碰撞检测
class CollisionSystem
    function update(world, deltaTime)
        # 简化的碰撞检测
        system.out.println("检测碰撞...")
    end
end

# 初始化游戏
var player = createPlayer(gameWorld, 100, 300)
for i = 0, i < 5, ++i
    createEnemy(gameWorld, 800 + i * 100, 300 + (i * 50))
end

# 游戏循环
var enemySystem = new EnemyMovementSystem{}
var collisionSystem = new CollisionSystem{}

for tick = 0, tick < 100, ++tick
    var dt = 0.016
    
    enemySystem.update(gameWorld, dt)
    collisionSystem.update(gameWorld, dt)
    
    runtime.delay(16)
end
```

