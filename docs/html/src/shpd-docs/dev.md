## 协作开发SHPD-API-CODE

若需要协作开发SHPD-API-CODE-DOXYGEN,  
请使用以下格式进行您的注释贡献。

开源地址：
https://github.com/LingASDJ/SHPD-ADE-PACKAGE/tree/ADE-2.2.1



<pre>
/**
 *
 */

 	/**绘制圆形
	 * 使用方法：<br>
	 * Painter.drawCircle(level, center, eyeRadius - 2, EMPTY_SP);<br>
	 * <br>
	 * Author:BinJie GPT<br>
	 * 注意：此代码使用AI生成<br>
	 * AIModel-Author：JDSA-Ling<br>
	 * @param level 楼层
	 * @param center 中心点
	 * @param radius 半径
	 * @param terrain 所需要的地形
	 */
</pre>


```java
	/**绘制圆形
	 * 使用方法：<br>
	 * Painter.drawCircle(level, center, eyeRadius - 2, EMPTY_SP);<br>
	 * <br>
	 * Author:BinJie GPT<br>
	 * 注意：此代码使用AI生成<br>
	 * AIModel-Author：JDSA-Ling<br>
	 * @param level 楼层
	 * @param center 中心点
	 * @param radius 半径
	 * @param terrain 所需要的地形
	 */
	public static void drawCircle(Level level, Point center, int radius, int terrain) {
		int cx = center.x;
		int cy = center.y;
		for (int x = cx - radius; x <= cx + radius; x++) {
			for (int y = cy - radius; y <= cy + radius; y++) {
				int dx = x - cx;
				int dy = y - cy;
				if (dx * dx + dy * dy <= radius * radius) {
					Painter.set(level, x, y, terrain);
				}
			}
		}
	}
```
 并提出你的PR,我们会进行审核考虑是否合并。

 2024-1-7 第一版



