v1:
�����ں� drivers/char
�޸� drivers/char/Makefile����ӣ�
obj-y += myleds_4412.o

���±����ں�

v2:
�� leds_4412.c �ŵ�drivers/leds
�޸� drivers/leds/Makefile:
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

ʵ�飺
�������岦��SD����������д�ں˵������壬����nand����������������
# echo 255 > /sys/class/leds/led1/brightness   ����������led1��
��˸
echo timer > /sys/class/leds/led1/trigger
echo 100 > /sys/class/leds/led1/delay_on
echo 200 > /sys/class/leds/led1/delay_off

�ر�
echo 0 > /sys/class/leds/led1/delay_on
��
echo 0 > /sys/class/leds/led1/brightness

�ϴ����ΰ汾���뵽GitHub��
1.git add -A
2.git commit -m "v2, use led class"
3.git tag v2
4.git push origin master
5.git push origin --tags



2. Linux��led class����
android-5.0.2\hardware\libhardware\include\hardware\lights.h

echo 255 > /sys/class/leds/led1/brightness
cat /sys/class/leds/led1/brightness
cat /sys/class/leds/led1/max_brightness

��˸
echo timer > /sys/class/leds/led1/trigger
echo 100 > /sys/class/leds/led1/delay_on
echo 200 > /sys/class/leds/led1/delay_off

�ر�
echo 0 > /sys/class/leds/led1/delay_on
��
echo 0 > /sys/class/leds/led1/brightness


������˸����:
echo timer > /sys/class/leds/led1/trigger  // timer��Ӧ ledtrig-timer.c

led_trigger_store // 1. ��trigger_list�ҳ���Ϊ"timer"��trigger
	list_for_each_entry(trig, &trigger_list, next_trig) {
		if (!strcmp(trigger_name, trig->name)) {
		    // 2. ����
		    led_trigger_set(led_cdev, trig);
		        // 3. ��trigger����led_classdev��trig_list������
		        list_add_tail(&led_cdev->trig_list, &trigger->led_cdevs);
		        led_cdev->trigger = trigger;
		        // 4. 
		        trigger->activate(led_cdev);
		           // 5. ����"timer"
		           timer_trig_activate
		               // 6. ����2���ļ�: delay_on, delay_off
		               device_create_file
		               device_create_file
		               led_blink_set // ��LED��˸
		               		led_set_software_blink
		               				
		               
		}
	}


echo 100 > /sys/class/leds/led1/delay_on
led_delay_on_store
  state = simple_strtoul(buf, &after, 10);
	led_blink_set  // // ��LED��˸
	led_cdev->blink_delay_on = state;

echo 200 > /sys/class/leds/led1/delay_off
led_delay_off_store
	state = simple_strtoul(buf, &after, 10);
	led_blink_set // ��LED��˸
	led_cdev->blink_delay_off = state;





��ôд������
a1. ����led_classdev
a2. ���� �� 
led_cdev->max_brightness
led_cdev->brightness_set
led_cdev->flags
led_cdev->brightness
led_cdev->name