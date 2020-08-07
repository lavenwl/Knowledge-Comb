### 全局唯一ID的实现方式

1. UUID, 32个字符, 4个横线

    >  `8 - 4 - 4 - 4 - 12`

    * 基于时间的UUID(time & mac)
    * 基于名字的UUID
    * 随机ID

2. 通过建立一个ID序列表实现统一ID

    ```sql
    create table sequence(
    	id int auto_increment
      b_id int unique_key
    )
    
    begin
    	replace into sequence(b_id) values();
    	select LAST_INSERT_ID();
    commit;
    ```

3. 利用第三方库如Redis(increBy/incr), mongdb(ObjectId)

4. 雪花算法

    ```java
    package com.laven.idgenerator;
    
    public class SnowFlakeGenerator {
        // 机房ID
        private long roomId;
        // 机器ID
        private long workerId;
        // 递增序列开始的值
        private long sequence;
    
        // 机房ID占用 5 个bit位
        private long roomIdBit = 5L;
        // 机器ID占用 5 个bit位
        private long workerIdBit = 5L;
        // 递增序列占用 12 个bit位
        private long sequenceBits = 12L;
    
        // 机房ID的最大值 TODO 这里与直接写32有什么区别
        private long maxRoomId = -1L ^ (-1L << roomIdBit);
        // 机器ID的最大值
        private long maxWorkerId = -1L ^ (-1L << workerIdBit);
        // 递增序列的最大值
        private long maxSequence = -1L ^ (-1L << sequenceBits);
    
        // 上一次ID生成的时间
        private long lastTimeStamp = -1L;
        // 初始时间值 TODO 这个什么用?
        private long twepoch = 1596372483166L;
    
        // 机器ID偏移量
        private long workerIdShift = sequenceBits;
        // 机房ID偏移量
        private long roomIdShift = sequenceBits + workerIdBit;
        // 时间数据偏移量
        private long timeStampShift = sequenceBits + workerIdBit + roomIdBit;
    
        public SnowFlakeGenerator(long roomId, long workerId, long sequence) {
            if (workerId > maxWorkerId || workerId < 0) {
                throw new IllegalArgumentException("worker Id 错误");
            }
            if (roomId > maxRoomId || roomId < 0) {
                throw new IllegalArgumentException("room Id 错误");
            }
            this.roomId = roomId;
            this.workerId = workerId;
            this.sequence = sequence;
        }
    
        public synchronized long nextVal() {
            long timestamp = System.currentTimeMillis();
            if (timestamp < lastTimeStamp) {
                throw new RuntimeException("时间戳异常");
            }
            if (lastTimeStamp == timestamp) {
                sequence = (sequence + 1) & maxSequence;
                if (sequence == 0) {
                    // sequence超过最大值(4095)
                    timestamp = waitToNextMills(lastTimeStamp);
                }
            } else {
                // TODO 技巧 如果进入到新的时间毫秒数, 置零处理
                sequence = 0;
    //            sequence = timestamp & 1;
            }
    
            lastTimeStamp = timestamp;
            return (
                    (timestamp - twepoch) << timeStampShift |
                    (roomId << roomIdShift) |
                    (workerId << workerIdShift) |
                    sequence
            );
        }
    
        private long waitToNextMills(long lastTimeStamp) {
            long timestamp = System.currentTimeMillis();
            while (timestamp <= lastTimeStamp) {
                timestamp = System.currentTimeMillis();
            }
            return timestamp;
        }
    
        public static void main(String[] args) throws InterruptedException {
            long timestamp = System.currentTimeMillis();
    //        System.out.println(timestamp);
    //        System.out.println(timestamp&1);
    //        System.out.printf(Long.toBinaryString(timestamp) + " 长度:" + Long.toBinaryString(timestamp).length());
            SnowFlakeGenerator snowFlakeGenerator = new SnowFlakeGenerator(1, 1, 1);
            for (int i = 0; i < 100; i++) {
    //            Thread.sleep(1);
                System.out.println(snowFlakeGenerator.nextVal());
            }
        }
    
    }
    
    ```

5. 美团(leaf算法)

