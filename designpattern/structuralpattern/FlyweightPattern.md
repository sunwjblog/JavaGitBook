# 享元模式

享元模式主要通过对象的复用来减少对象创建的次数和数量，以减少系统内存的使用和降低系统的负载。享元模式属于结构型模式，在系统需要一个对象时享元模式首先在系统中查找并尝试重用现有的对象，如果未找到匹配的对象，则创新新对象并将其缓存在系统中以便下次使用。

享元模式主要用于避免在有大量对象时频繁创建和销毁对象造成系统资源的浪费，把其中共同的部分抽象出来，如果有相同的业务请求，则直接返回内存中已有的对象，避免重新创建。

如类图：

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/享元模式类图.png)

### 示例代码

* 定义Memory

```java
@Setter
@Getter
@ToString
public class Memory {

    private int size;
    private boolean isused;
    private String id;

    public Memory(int size, boolean isused, String id) {
        this.size = size;
        this.isused = isused;
        this.id = id;
    }
}
```

* 定义MemoryFactory工厂

```java
public class MemoryFactory {

    private static List<Memory> memoryList = new ArrayList<>();

    public static Memory getMemory(int size) {
        Memory memory = null;
        for (int i = 0; i < memoryList.size(); i++) {
            memory = memoryList.get(i);
            // 如果存在和需求size一样大小并且未使用的内存块，则直接返回
            if (memory.getSize() == size && memory.isIsused() == false) {
                memory.setIsused(true);
                memoryList.set(i,memory);
                System.out.println("get memory form memoryList:" + memory);
                break;
            }
        }

        // 如果内存不存在，则从系统中申请新的内存返回，并将该内存加入内存对象列表中
        if (memory == null) {
            memory = new Memory(32, false, UUID.randomUUID().toString());
            System.out.println("create a new memory form system and add to memoryList: " + memory);
            memoryList.add(memory);
        }
        return memory;
    }

    public static void releaseMemory(String id) {
        for (int i = 0; i < memoryList.size(); i++) {

            Memory memory = memoryList.get(i);
            // 如果存在和需求size一样大小并且未使用的内存块，则直接返回
            if (memory.getId().equals(id)) {
                memory.setIsused(false);
                memoryList.set(i, memory);
                System.out.println("release memory: " + id);
                break;
            }

        }
    }
}
```

* 使用享元模式

```java
public class FlaweightTest {

    public static void main(String[] args) {

        // 首次获取内存，将创建一个内存
        Memory memory = MemoryFactory.getMemory(32);
        // 在使用后释放内存
        MemoryFactory.releaseMemory(memory.getId());
        // 重新获取内存
        MemoryFactory.getMemory(32);
    }
}
```

