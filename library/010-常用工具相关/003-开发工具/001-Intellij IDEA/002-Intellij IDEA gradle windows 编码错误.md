# Intellij IDEA Gradle windows编码问题

使用Intellij在Windows中编译gradle时，会遇到编码问题，此时可以点击build.gradle左边的绿色运行按钮，选择“Edit ***”,在弹出窗口的VM options中填入`-Dfile.encoding=UTF-8`即可。