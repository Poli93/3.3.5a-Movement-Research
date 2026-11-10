### 3.3.5a client movement updates and distance checks

**Introduction**

The server knows where you are only from movement packets your client sends. While you move, there is a movement heartbeat every 250ms (Blizzlike 500ms), plus an instant packet on every movement action: key press, key release, jump, landing and every 0.1 radians (5.7°) of mouse turn from the facing in your last movement packet of any kind. 

**Distance Formulas**

```
Melee attack: MAX(5, SourceCombatReach + TargetCombatReach + 4/3) + IfSourceAndTargetAreAndRunning(8/3)
Ranged attack: SourceCombatReach + TargetCombatReach + AttackRange + IfSourceAndTargetAreAndRunning(8/3)
AoE: TargetCombatReach + AttackRadius + IfSourceIsRunning(8/3) (CombatReach applies only to a center-to-edge ability, does not apply to center-to-center abilities)
```

**Myth: Jumping takes longer to update: FALSE**

Jumping updates faster than running, contrary to common belief!, a jump takes 0.825 seconds

```AirTime = 2 * launch speed / gravity = 2 * 7.9555 / 19.2911 = 0.8248s```

While midair, the client still sends movement heartbeats, not only, the client also sends **more** updates while adding jumping, as the **more** movement actions the more updates.

Examples:

_straight line running, client sends heartbeat updates every 250ms_
```
11/04 20:40:23.421   Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11769.338 -1673.278 52.919 o=0.024  moving=1 flags=0x1
11/04 20:40:23.673   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11767.448 -1673.233 52.922 o=0.024  moving=1 flags=0x1
11/04 20:40:23.921   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11765.559 -1673.187 52.926 o=0.024  moving=1 flags=0x1
11/04 20:40:24.173   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11763.670 -1673.141 52.930 o=0.024  moving=1 flags=0x1
11/04 20:40:24.421   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11761.780 -1673.096 52.934 o=0.024  moving=1 flags=0x1
11/04 20:40:24.673   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11759.891 -1673.050 52.938 o=0.024  moving=1 flags=0x1
11/04 20:40:24.922   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11758.001 -1673.004 52.942 o=0.024  moving=1 flags=0x1
11/04 20:40:25.171   Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11756.111 -1672.959 52.943 o=0.024  moving=1 flags=0x1
11/04 20:40:25.203   Roguealah#B8548D  MSG_MOVE_STOP C2S  -11755.870 -1672.953 52.943 o=0.024  moving=0 flags=0x0
```
_straight line running, adding jumps_
```
11/04 21:46:39.508  Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11820.824 -1677.633 52.947 o=0.052  moving=1 flags=0x1
11/04 21:46:39.759  Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11818.937 -1677.536 52.947 o=0.052  moving=1 flags=0x1
11/04 21:46:40.011  Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11817.049 -1677.438 52.944 o=0.052  moving=1 flags=0x1
11/04 21:46:40.209  Roguealah#B8548D  MSG_MOVE_JUMP C2S  -11815.532 -1677.359 52.939 o=0.052  moving=1 flags=0x1001
11/04 21:46:40.460  Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11813.645 -1677.262 54.325 o=0.052  moving=1 flags=0x1001
11/04 21:46:40.709  Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11811.757 -1677.164 54.506 o=0.052  moving=1 flags=0x1001
11/04 21:46:40.959  Roguealah#B8548D  MSG_MOVE_HEARTBEAT C2S  -11809.870 -1677.066 53.480 o=0.052  moving=1 flags=0x1001
11/04 21:46:41.035  Roguealah#B8548D  MSG_MOVE_FALL_LAND C2S  -11809.292 -1677.036 52.926 o=0.052  moving=1 flags=0x1
```
_straight line running, adding jumps and spamming movement keys midair_
```
11/04 20:39:51.254  Roguealah#B8548D  MSG_MOVE_JUMP C2S  -11814.762 -1676.197 52.939 o=0.005  moving=1 flags=0x1001
11/04 20:39:51.334  Roguealah#B8548D  MSG_MOVE_STOP C2S  -11814.157 -1676.194 53.514 o=0.005  moving=1 flags=0x5001
11/04 20:39:51.532  Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11812.660 -1676.188 54.406 o=0.005  moving=1 flags=0x1001
11/04 20:39:51.624  Roguealah#B8548D  MSG_MOVE_STOP C2S  -11811.965 -1676.185 54.563 o=0.005  moving=1 flags=0x5001
11/04 20:39:51.703  Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11811.367 -1676.182 54.567 o=0.005  moving=1 flags=0x1001
11/04 20:39:51.782  Roguealah#B8548D  MSG_MOVE_STOP C2S  -11810.771 -1676.179 54.451 o=0.005  moving=1 flags=0x5001
11/04 20:39:51.884  Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11809.999 -1676.176 54.123 o=0.005  moving=1 flags=0x1001
11/04 20:39:51.955  Roguealah#B8548D  MSG_MOVE_STOP C2S  -11809.462 -1676.173 53.777 o=0.005  moving=1 flags=0x5001
11/04 20:39:52.044  Roguealah#B8548D  MSG_MOVE_START_FORWARD C2S  -11808.789 -1676.170 53.205 o=0.005  moving=1 flags=0x1001
11/04 20:39:52.082  Roguealah#B8548D  MSG_MOVE_FALL_LAND C2S  -11808.512 -1676.169 52.924 o=0.005  moving=1 flags=0x1
```
