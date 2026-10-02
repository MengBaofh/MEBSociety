# MEBSociety v2.1.0 更新日志

---

## 🎉 重大更新：NBT 自定义物品商店支持

MEBSociety 2.1.0 版本带来了商店系统的重大升级，现在完美支持 NBT 自定义物品！

---

## ✨ 新增功能

### 1. 商店支持 NBT 物品

商店现在可以正确存储和还原带有 NBT 数据的自定义物品：

- ✅ **完整保存物品属性**：名称、描述、附魔、特殊效果、耐久度等
- ✅ **购买即可使用**：玩家购买后获得功能完整的自定义物品
- ✅ **与 MEBCustomItem 完美集成**：自动注册自定义物品到商店

### 2. API 增强

#### `addItemShop()` 方法升级

```php
public function addItemShop(
    string $name, 
    string $itemName, 
    int $num, 
    float $buyPrice, 
    float $sellPrice, 
    ?Item $customItem = null  // 新增：可选的自定义物品参数
): int
```

**使用示例**：

```php
// 基础物品（旧方式，仍然支持）
$shop->addItemShop("钻石", "minecraft:diamond", 1, 1000, 500);

// 自定义物品（新方式）
$customItem = $itemManager->createItem("flame_sword");
$shop->addItemShop(
    "§c§l烈焰之剑",           // 显示名称
    "minecraft:diamond_sword", // 基础物品
    1,                         // 数量
    10000,                     // 购买价格
    5000,                      // 出售价格
    $customItem                // 自定义物品实例
);
```

#### 新增 `removeItemShop()` 方法

```php
public function removeItemShop(string $itemName): bool
```

用于根据物品名称删除商店中的商品（主要用于 MEBCustomItem）：

**使用示例**：

```php
// 删除商店中的自定义物品
$shop->removeItemShop("mebci:flame_sword");
// 返回 true 表示成功删除，false 表示未找到
```

**应用场景**：
- MEBCustomItem 删除自定义物品时，自动从商店移除
- 防止玩家购买已删除的物品
- 保持商店与自定义物品的同步

#### `getShopItem()` 方法增强

现在支持从 NBT 数据还原自定义物品：

```php
$item = $shop->getShopItem($shopId);
// 返回的物品包含所有自定义属性
```

---

## ⚠️ 注意事项

### 向后兼容

- ✅ 完全兼容旧版本的商店配置
- ✅ 不传入 `$customItem` 参数时，行为与 2.0.9 版本一致
- ✅ 现有商品无需修改，自动兼容

### 功能限制

- ❌ **自定义物品暂不支持出售**：由于 NBT 数据比对复杂，玩家无法将自定义物品卖回商店
- ⚠️ **自定义物品自动设置为 `可出售: false`**

---

## 📝 修改文件

- `src/MengBao/MEBSociety/Units/Shop.php`
  - `addItemShop()` - 新增 `$customItem` 参数
  - `getShopItem()` - 新增 NBT 数据还原逻辑

---

## 💡 开发者指南

### 注册自定义物品到商店

```php
use MengBao\MEBSociety\Main as MEBSociety;

// 获取 MEBSociety 实例
$society = $this->getServer()->getPluginManager()->getPlugin("MEBSociety");
if ($society instanceof MEBSociety) {
    $shop = $society->getShopUnit();
    
    // 创建自定义物品
    $item = VanillaItems::DIAMOND_SWORD();
    $item->setCustomName("§c§l我的自定义剑");
    $item->setLore(["§7这是一把特殊的剑"]);
    $item->addEnchantment(new EnchantmentInstance(VanillaEnchantments::SHARPNESS(), 5));
    
    // 注册到商店
    $shop->addItemShop(
        "§c§l我的自定义剑",
        "minecraft:diamond_sword",
        1,
        5000,
        2500,
        $item  // 传入自定义物品实例
    );
}
```