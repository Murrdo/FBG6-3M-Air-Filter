# FBG6-3M-Air-Filter
Air filter for FBG6

Фильтр для принтера FBG6 испоьзующий сменные фильтры 3M (проверено на 6059) - при условии общей герметизации запаха нет при 10 часовой печати в закрытой непроветриваемой комнате.

Подготовка и принцип действия.
Фильтр работает при принципу "подпора" внешним давлением, т.е. во время работы вентилятор создаёт минимальное разряжение внутри, что бы через щели (которые в любом случае остануться) воздух снаружи поступал внутрь, а не выходил наружу. При этом ущерба темпиратуре в камере он не наносит.
Фильтр обазтельно требует актировый колпак.
Что необходимо проклеить виброй или любым другим удобным материалом:
1.Кромку примыкания дверь и корпуса (очень мягкая вспененая резина толщиной 3мм)
2.Колпак по периметру примыкания к корпусу (та же резина)
3.Отсек с электроникой
4.Щели на дне камеры
5.Все отверстия под хомуты, под провод на датчик филамента), в общем вообще все мельчашие вещи, все что видите.

В норме запах наружу не попадает, т.е. например вплотную носом не ощущается запах у отверстия под трубку подачи филамента. Но при этом поток на выходу фильтра едва ощутим рукой.

Установка самого фильтра:
Внутренний блок под вентилятор 5015 обязательно перед установкой прокилеить по кромке примыкания к корпусу мягкий двхсторонним скотчем, вентялтор можно посадить на клей типа б7000 (хотя это излишне). Вентилятор вкручивается болтами на 3 вплавляемые втулки.
Наружный блок так же по периметру примыкания к корпсу на скотч. Крепление предфильтра 3м прикручивается на 6 болтов м3*5 в закладные, под него обазтельно нанести любой герметик по всей площади (B7000 прекрасно подойдёт) примыкания к креплению.

Пример организация управления фильтром, стартовый код Prusaslicer/Superslicer:
```
START_PRINT  EXTRUDER_TEMP={first_layer_temperature[0]} BED_TEMP={first_layer_bed_temperature[0]} MATERIAL={filament_type[0]}
```
Макрос и конфиг в klipper:

```
[fan_generic Filter_Fan]
pin: PD15
kick_start_time = 0.8
cycle_time = 0.025
```
Включение если идёт печать АБСом
```
[gcode_macro START_PRINT]
description = Start routine for the print
variable_retract = 6

gcode = 
	{% set extruder_temp = params.EXTRUDER_TEMP|default(240)|float %}
	{% set travel_temp = params.TRAVEL_TEMP|default(extruder_temp - 50 )|float %}
	{% set bed_temp = params.BED_TEMP|default(70)|float %}
	{% set MATERIAL = params.MATERIAL|default('PLA')|string %}
	CLEAR_PAUSE
	M220 S100 #сброс скорости печати на 100
	M221 S100
	G90 #установка абсолютных координат
	M82 #экструдер в абсолютный режим
	SET_HEATER_TEMPERATURE HEATER=extruder TARGET={travel_temp}
	SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET={bed_temp}
	SET_FAN_SPEED FAN=Filter_Fan SPEED=0
	{% if MATERIAL == 'ABS' %}
		filter_on
    {% endif %}
	G28
	TEMPERATURE_WAIT SENSOR=heater_bed MINIMUM={bed_temp}
	SET_HEATER_TEMPERATURE HEATER=extruder TARGET={extruder_temp}
	TEMPERATURE_WAIT SENSOR=extruder MINIMUM={extruder_temp}	
```
