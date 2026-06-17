# 代码对齐规则

0. 此处所指代码对齐，不是指缩进对齐，而是指额外空格的对齐

## 默认规则

   * 虽然有时在代码中添加空格进行列对齐在视觉上有一定作用，但由于维护困难，应当谨慎添加额外的空格强行对齐。
   * 通常行尾注释与代码之间，代码的token和token之间最多保留一个空格。
   * 不为了“视觉整齐”在普通语句中插入多余空格。
不好的例子：
        x = self.pool(F.relu(self.conv1(x)))   # -> B x A x H1 x W1
        x = self.pool(F.relu(self.conv2(x)))   # -> B x B x H2 x W2
        x = x.view(x.size(0), -1)              # 展平: B x (B*H2*W2)
        x = self.fc1(x)                        # -> B x 25
后两行为了对齐上文，代码和注释间用了数不清的空格，这是没有必要的。
推荐：

   ```python
   x = self.pool(F.relu(self.conv1(x))) # -> B x C1 x H1 x W1
   x = self.pool(F.relu(self.conv2(x))) # -> B x C2 x H2 x W2
   x = x.view(x.size(0), -1) # 展平: B x (C2*H2*W2)
   x = self.fc1(x) # -> B x 25
   ```

## 允许的例外
   在以下情况，可以使用对齐：

   * 多行**结构完全一致**（如 switch/case、大量变量初始化、表驱动映射、配置列表）
   * 右侧表达式形态一致（例如都是构造对象或简单赋值）
   * 代码很可能被**批量修改**（例如统一重命名变量）

   示例（允许）：

   ```cpp
   case NoteType::TAP:   sprite = new TapNoteSprite();   break;
   case NoteType::HOLD:  sprite = new HoldNoteSprite();  break;
   case NoteType::TOUCH: sprite = new TouchNoteSprite(); break;
   ```

## 限制范围（防止滥用）

   * 对齐只允许出现在**局部连续代码块**中，不得扩散到整个函数或文件
   * 不允许跨不同语句类型对齐（例如把声明、赋值、调用混在一起对齐）
   * 一旦结构不再一致，应立即停止对齐

## 优先级原则

   * 可读性优先于对齐
   * 简洁性优先于整齐
   * 若不确定是否应对齐，则不要对齐

> 以上内容仅针对需要额外空格进行对齐的情况，代码缩进依然按照最佳实践和规范进行书写。