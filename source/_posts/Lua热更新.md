---
title: Lua热更新
date: 2016-09-17 16:53:51
tags:
    - Lua
---

## 什么是热更新

> Hot swapping is ability to alter the running code of a program without needing to interrupt its execution.
                                    --[Wikipedia](https://en.wikipedia.org/wiki/Hot_swapping)

热更新: lua虚拟机运行时, 修改出现bug或者想要增加新feature的代码, 不需要去重启整个服务.

<!--more-->


## require热更新方案

require的原理:

1. 在registry["_LOADED"]表中判断该模块是否已经加载过了, 如果是则直接换回, 避免重复加载某个模块代码
2. 一次调用注册的loader来加载模块
3. 将加载过的模块赋值给registry["LOADER"]表

require实现的缓存机器会阻止重复加载相同模块. 所以使用require可以把`package.loaded`中对应的模块设置为nil, 然后重新进行require加载

- 缺点: 数据会被同时更新(全局变量), 这并不是我们想要的

```lua
-- 1. 清除require缓存, 来实现热加载
function require_ex(module_name)
    print(string.format("require_ex = %s", module_name))
    if package.loaded[module_name] then
        print(string.format("require_ex module[%s] reload.", module_name))
    end
    package.loaded[module_name] = nil  -- 清除历史, 重新load
    return require(module_name)
end
```

## 优化的热更新方案

upvalue被保留在热更新中非常重要, `upvalue`为**函数里用到的定义在该函数之前的local变量**

优化的热更新方案主要是将upvalue保存下来, 重新加载时, 把upvalue的值更新进去, 使之整体看起来与原来一样.

```lua
-- 2. 优化后的热加载
-- 利用_ENV环境，在加载的时候把数据加载到_ENV下，然后再通过对比的方式修改_G底下的值，从而实现热更新

function hot_swap(chunk, check_name)
    local env = {}
    setmetatable(env, { __index = _G })
    local _ENV = env
    local f, err = load(chunk, check_name,  't', env)
    assert(f, err)
    local ok, err = pcall(f)
    assert(ok, err)
    for name, value in pairs(env) do
        local g_value = _G[name]
        -- 原type和新类型不同则做覆盖, 否则保持原值
        if type(g_value) ~= type(value) then
            _G[name] = value
        elseif type(value) == 'function' then
            update_func(value, g_value, name, 'G'..'  ')
            _G[name] = value
        elseif type(value) == 'table' then
            update_table(value, g_value, name, 'G'..'  ')
        end
    end
end

function update_func(env_f, g_f, name, deep)
    -- 取得原值所有的upvalue，保存起来
    local old_upvalue_map = {}
    for i = 1, math.huge do
        local name, value = debug.getupvalue(g_f, i)
        if not name then break end
        old_upvalue_map[name] = value
    end
    -- 遍历所有新的upvalue，根据名字和原值对比，如果原值不存在则进行跳过，如果为其它值则进行遍历env类似的步骤
    for i = 1, math.huge do
        local name, value = debug.getupvalue(env_f, i)
        if not name then break end
        local old_value = old_upvalue_map[name]
        if old_value then
            if type(old_value) ~= type(value) then
                debug.setupvalue(env_f, i, old_value)
            elseif type(old_value) == 'function' then
                update_func(value, old_value, name, deep..'  '..name..'  ')
            elseif type(old_value) == 'table' then
                update_table(value, old_value, name, deep..'  '..name..'  ')
                debug.setupvalue(env_f, i, old_value)
            else
                debug.setupvalue(env_f, i, old_value)
            end
        end
    end
end

local protection = {
    setmetatable = true,
    pairs = true,
    ipairs = true,
    next = true,
    require = true,
    _ENV = true,
}
-- 防止重复的table替换，造成死循环
local visited_sig = {}
function update_table(env_t, g_t, name, deep)
    -- 对某些关键函数不进行比对
    if protection[env_t] or protection[g_t] then return end
    --如果原值与当前值内存一致，值一样不进行对比
    if env_t == g_t then return end
    local signature = tostring(g_t)..tostring(env_t)
    if visited_sig[signature] then return end
    visited_sig[signature] = true
    -- 遍历对比值，如进行遍历env类似的步骤
    for name, value in pairs(env_t) do
        local old_value = g_t[name]
        if type(value) == type(old_value) then
            if type(value) == 'function' then
                update_func(value, old_value, name, deep..'  '..name..'  ')
                g_t[name] = value
            elseif type(value) == 'table' then
                update_table(value, old_value, name, deep..'  '..name..'  ')
            end
        else
            g_t[name] = value
        end
    end
    --遍历table的元表，进行对比
    local old_meta = debug.getmetatable(g_t)
    local new_meta = debug.getmetatable(env_t)
    if type(old_meta) == 'table' and type(new_meta) == 'table' then
        update_table(new_meta, old_meta, name..'s Meta', deep..'  '..name..'s Meta'..'  ' )
    end
end


function hot_reload(name)
    local file_str
    local fp = io.open(name)
    if fp then
        io.input(name)
        file_str = io.read('*all')
        io.close(fp)
    end

    if not file_str then
        return -1
    end
    return hot_swap(file_str, name)
end
```


## 参考链接

- [Lua 5.2/5.3 热更新小结](http://www.jianshu.com/p/4fca228240c0)
