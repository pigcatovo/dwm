# DWM教程

## 脚本

#### 更换壁纸

wp-change.sh

 ```
sudo pacman -S feh
 ```

```sh
#!/bin/bash
feh --recursive --randomize --bg-fill ~/Pictures/wallpapers/ghibili
```

#### 自动更换壁纸

wp-autochange.sh

```sh
#!/bin/bash
while true; do
	/bin/bash ~/scripts/wp-change.sh
	sleep 3m
done
```

 #### 声音增大

vol-up.sh

```sh
#!/bin/bash
/usr/bin/amixer -qM set Master 5%+ umute
bash ~/scripts/dwm-status-refresh.sh
```

 #### 声音减小

vol-down.sh

```sh
#!/bin/bash
/usr/bin/amixer -qM set Master 5%- umute
bash ~/scripts/dwm-status-refresh.sh
```

 #### 声音禁用

vol-toggle.sh

```sh
#!/bin/bash
/usr/bin/amixer set Master toggle
bash ~/scripts/dwm-status-refresh.sh
```

#### 触摸板点击

tap-to-click.sh

```
sudo pacman -S xorg-xinput
```



```sh
#!/bin/bash

# Get id of touchpad and the id of the field corresponding to
# tapping to click
id=`xinput list | grep "Touchpad" | cut -d'=' -f2 | cut -d'[' -f1`
tap_to_click_id=`xinput list-props $id | \
                      grep "Tapping Enabled (" \
                      | cut -d'(' -f2 | cut -d')' -f1`

# Set the property to true
xinput --set-prop $id $tap_to_click_id 1
```

 #### 触摸板自然滚动

```sh
#!/bin/bash
id=`xinput list | grep "Touchpad" | cut -d'=' -f2 | cut -d'[' -f1`
natural_scrolling_id=`xinput list-props $id | \
                      grep "Natural Scrolling Enabled (" \
                      | cut -d'(' -f2 | cut -d')' -f1`
# Set the property to true
xinput --set-prop $id $natural_scrolling_id 1
```

####  进入暂停模式

suspend.sh

```sh
#!/bin/bash
systemctl suspend
```

*suspend：暂停模式会将系统的状态保存到内存中，然后关闭掉大部分的系统硬件*

#### 状态栏 (自动刷新)

dwm-status.sh

```sh
#!/bin/bash
while true
do
	bash ./dwm-status-refresh.sh
	sleep 2
done
```

#### 状态栏刷新

dwm-status-refresh.sh

```sh
#!/bin/bash

function get_bytes {
	# Find active network interface
	interface=$(ip route get 8.8.8.8 2>/dev/null| awk '{print $5}')
	line=$(grep $interface /proc/net/dev | cut -d ':' -f 2 | awk '{print "received_bytes="$1, "transmitted_bytes="$9}')
	eval $line
	now=$(date +%s%N)
}

# Function which calculates the speed using actual and old byte number.
# Speed is shown in KByte per second when greater or equal than 1 KByte per second.
# This function should be called each second.

function get_velocity {
	value=$1
	old_value=$2
	now=$3

	timediff=$(($now - $old_time))
	velKB=$(echo "1000000000*($value-$old_value)/1024/$timediff" | bc)
	if test "$velKB" -gt 1024
	then
		echo $(echo "scale=2; $velKB/1024" | bc)MB/s
	else
		echo ${velKB}KB/s
	fi
}

# Get initial values
get_bytes
old_received_bytes=$received_bytes
old_transmitted_bytes=$transmitted_bytes
old_time=$now

print_volume() {
	volume="$(amixer get Master | tail -n1 | sed -r 's/.*\[(.*)%\].*/\1/')"
	if test "$volume" -gt 0
	then
		echo -e "vol.${volume}"
	else
		echo -e "mute"
	fi
}

print_mem(){
	memfree=$(($(grep -m1 'MemAvailable:' /proc/meminfo | awk '{print $2}') / 1024))
	echo -e "$memfree"
}

print_temp(){
	test -f /sys/class/thermal/thermal_zone0/temp || return 0
	echo $(head -c 2 /sys/class/thermal/thermal_zone0/temp)C
}

get_battery_combined_percent() {

	# get charge of all batteries, combine them
	total_charge=$(expr $(acpi -b | awk '{print $4}' | grep -Eo "[0-9]+" | paste -sd+ | bc));

	# get amount of batteries in the device
	battery_number=$(acpi -b | wc -l);

	percent=$(expr $total_charge / $battery_number);

	echo $percent;
}

get_battery_charging_status() {

	if $(acpi -b | grep --quiet Discharging)
	then
		echo "B:";
	else 
		echo "C";
	fi
}

print_bat(){
	echo "$(get_battery_charging_status) $(get_battery_combined_percent)";
}

print_date(){
	date '+%a %m-%d %H:%M'
}

show_record(){
	test -f /tmp/r2d2 || return
	rp=$(cat /tmp/r2d2 | awk '{print $2}')
	size=$(du -h $rp | awk '{print $1}')
	echo " $size $(basename $rp)"
}


LOC=$(readlink -f "$0")
DIR=$(dirname "$LOC")
export IDENTIFIER="unicode"



get_bytes

# Calculates speeds
vel_recv=$(get_velocity $received_bytes $old_received_bytes $now)
vel_trans=$(get_velocity $transmitted_bytes $old_transmitted_bytes $now)

xsetroot -name "  M:$(print_mem)M D:$vel_recv U:$vel_trans $(print_volume) $(print_bat) $(print_date) "

# Update old values to perform new calculations
old_received_bytes=$received_bytes
old_transmitted_bytes=$transmitted_bytes
old_time=$now

exit 0
```

#### 自动重复执行（delay）

autostart_wait.sh

```sh
#!/bin/bash
sleep 10
#fcitx &
```

#### 自动启动

autostart.sh

```
sudo pacman -S picom
sudo pacman -S network-manager-applet
sudo pacman -S xfce4-power-manager
```

```sh
#!/bin/bash
#状态栏
/bin/bash ~/scripts/dwm-status.sh &
#自动更换壁纸
/bin/bash ~/scripts/wp-autochange.sh &
#窗口渲染器
picom -b
#触摸板轻触点击
/bin/bash ~/scripts/tap-to-click.sh &
#触摸板自然滚动
/bin/bash ~/scripts/inverse-scroll.sh &
#WIFI管理器
nm-applet &
#电源管理
xfce4-power-manager &
#自动重复执行（delay）
#~/scripts/autostart_wait.sh &
```

## 配置文件

```
sudo pacman -S flameshot
```

快捷键：

