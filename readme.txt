v1:
放入内核 drivers/char
修改 drivers/char/Makefile，添加：
obj-y += myleds_4412.o

重新编译内核

v2:
把 leds_4412.c 放到drivers/leds
修改 drivers/leds/Makefile:
obj-y += leds_4412.o

make menuconfig

CONFIG_LEDS_CLASS
CONFIG_LEDS_TRIGGERS
CONFIG_LEDS_TRIGGER_TIMER

-> Device Drivers
  -> LED Support
   [*]   LED Class Support    
   [*]   LED Trigger support
   <*>   LED Timer Trigger

make zImage

实验：
将开发板拨到SD卡启动，烧写内核到开发板，拨到nand启动，启动开发板
# echo 255 > /sys/class/leds/led1/brightness   （用来点亮led1）
闪烁
echo timer > /sys/class/leds/led1/trigger
echo 100 > /sys/class/leds/led1/delay_on
echo 200 > /sys/class/leds/led1/delay_off

关闭
echo 0 > /sys/class/leds/led1/delay_on
或
echo 0 > /sys/class/leds/led1/brightness

上传本次版本代码到GitHub：
1.git add -A
2.git commit -m "v2, use led class"
3.git tag v2
4.git push origin master
5.git push origin --tags



2. Linux的led class驱动
android-5.0.2\hardware\libhardware\include\hardware\lights.h

echo 255 > /sys/class/leds/led1/brightness
cat /sys/class/leds/led1/brightness
cat /sys/class/leds/led1/max_brightness

闪烁
echo timer > /sys/class/leds/led1/trigger
echo 100 > /sys/class/leds/led1/delay_on
echo 200 > /sys/class/leds/led1/delay_off

关闭
echo 0 > /sys/class/leds/led1/delay_on
或
echo 0 > /sys/class/leds/led1/brightness


分析闪烁功能:
echo timer > /sys/class/leds/led1/trigger  // timer对应 ledtrig-timer.c

led_trigger_store // 1. 从trigger_list找出名为"timer"的trigger
	list_for_each_entry(trig, &trigger_list, next_trig) {
		if (!strcmp(trigger_name, trig->name)) {
		    // 2. 调用
		    led_trigger_set(led_cdev, trig);
		        // 3. 把trigger放入led_classdev的trig_list链表里
		        list_add_tail(&led_cdev->trig_list, &trigger->led_cdevs);
		        led_cdev->trigger = trigger;
		        // 4. 
		        trigger->activate(led_cdev);
		           // 5. 对于"timer"
		           timer_trig_activate
		               // 6. 创建2个文件: delay_on, delay_off
		               device_create_file
		               device_create_file
		               led_blink_set // 让LED闪烁
		               		led_set_software_blink
		               				
		               
		}
	}


echo 100 > /sys/class/leds/led1/delay_on
led_delay_on_store
  state = simple_strtoul(buf, &after, 10);
	led_blink_set  // // 让LED闪烁
	led_cdev->blink_delay_on = state;

echo 200 > /sys/class/leds/led1/delay_off
led_delay_off_store
	state = simple_strtoul(buf, &after, 10);
	led_blink_set // 让LED闪烁
	led_cdev->blink_delay_off = state;





怎么写驱动：
a1. 分配led_classdev
a2. 设置 ： 
led_cdev->max_brightness
led_cdev->brightness_set
led_cdev->flags
led_cdev->brightness
led_cdev->name