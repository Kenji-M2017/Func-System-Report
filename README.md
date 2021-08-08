# 機能システム学特論課題
講義内で実施したROSの演習課題

## 動作確認環境
* Ubuntu18.04(virtualbox)
* ROS Melodic

##使い方
### Rviz上での6自由度のマニピュレーションモデルの表示
実行する前に以下のコマンドでjoint-state-publisherのインストールを行う.
```
$ sudo apt install ros-melodic-joint-state-publisher-gui
```
以下のコマンドを実行する.
端末1
```
$ roslaunch sixdofarm sixdofarm.launch
```
端末2
```
$ rosrun rviz rviz
```
以下の手順を行うことで,Rviz上に6自由度マニピュレーションモデルが表示される.
1. 左のDisplayの欄にある「Fixed Frame」の「map」を「base」に変更する.
2. 「Add」のボタンを押して出てくるウィンドウ中の「By display type」タブの「RobotModel」を選択し,追加する.
joint_state_publisherのGUIの各項目を調整することで,6自由度マニピュレーションモデルの関節角が変化することを確認できる.

### Moveit上での6自由度のマニピュレーションモデルの制御
以下のコマンドを実行する.
端末
```
$ roslaunch sixdofarm_moveit_config demo.launch
```
6自由度マニピュレーションモデルの先端にあるボールのようなものをドラッグすることで姿勢を変化できる.
ドラッグした姿勢までの軌道の生成・実行を以下の手順で行う.
1. 「Planning」のタブを選び,「Commands」の下にある「Plan」をクリックする.
2. 「Execute」することで,計画した軌道を実行
計画と実行を同時に行う場合,「Plan&Execute」をクリックする.

### Gazebo上での6自由度のマニピュレーションモデルの表示
以下のコマンドを実行する.
端末
```
$ roslaunch sixdofarm_gazebo sixdofarm.launch
```
実行することで,Rvizで表示された6自由度のマニピュレーションモデルが表示される.

### 6自由度マニピュレーションモデルのros_controlでの制御
以下のコマンドを実行する.
端末1
```
$ roslaunch sixdofarm_gazebo sixdofarm.launch
```
Gazeboが起動した後,以下のコマンドを実行する.
端末2
```
$ roslaunch sixdofarm_control sixdofarm_control.launch
```
以下のコマンドで,「関節2」に「1.5rad」動かす指令を送る.
端末3
```
$ rostopic pub -1 /sixdofarm/joint2_position_controller/command std_msgs/Float64 "data: 1.5"
```

### rqtとdynamic reconfigureによるPID制御ゲインの調整
以下のコマンドを実行する.
端末1
```
$ roslaunch sixdofarm_gazebo sixdofarm.launch
```
Gazeboが起動した後,以下のコマンドを実行する.
端末2
```
$ roslaunch sixdofarm_control sixdofarm_control.launch
```
端末3
```
$ rqt
```
rqtで指令値を送るため,以下の手順を行う.
1. rqtの上部にある「Plugins」->「Topics」->「Message Publisher」を選択して追加する.
2. 「Topic」で操作したい関節のトピックを選択する.「type」は送信したい型を選択するだけで良い.「+」のボタンで追加する.
3. リストの表に入っている項目の「rate」の数字を変更することで,更新周波数を変更することができる.
4. 「expression」の項目に数式を書くことで,更新周波数に従って,状態を変化させることができる.
(例) 関節1の制御
「Topic」:/sixdofarm/joint1_position_controller/command
「Type」:std_msgs/Float64
「rate」:100
「expression」:sin(i/100*5)*1.7 + 0

これらの項目を変更することで,関節1が以下のパラメータに従って動作することをGazebo上で確認できる.
rate:100[hz]
speed:5
diff:1.7[rad]
offset:0[rad]

以下の手順で,動作の様子をrqtのモニターで確認する.
1. rqtの上部にある「Plugins」->「Visualization」->「Plot」を選択して追加する.
2. 「Topic」に追加したいトピック名を記載して,「+」のボタンで追加する.今回は3つのトピックを追加する.
* /sixdofarm/joint1_position_controller/command/data
* /sixdofarm/joint1_position_controller/state/error
* /sixdofarm/joint1_position_controller/state/process_value
3. rqtの上部にある「Plugins」->「Configuration」->「Dynamic Reconfigure」を選択して追加する.
4. 追加後,joint1のPIDゲインを調整し,所望する範囲に収まるように調整する.

### MoveitとGazeboの接続
以下のコマンドを実行する.
端末
```
$ roslaunch sixdofarm_moveit_config sixdofarm_moveit.launch
```
「Moveit上での6自由度のマニピュレーションモデルの制御」の項と同様に,軌道の生成・実行することで,Gazebo上で計画した軌道に従って動作することが確認できる.

### Gazebo上で構築した4輪移動ロボットモデルの表示
以下のコマンドを実行する.
端末
```
$ roslaunch wheel_robot wheel_robot.launch
```
実行することで,gazeboが立ち上がり,4輪の移動ロボットが表示される.
また,Rvizを立ち上げ,robot modelとTFを追加することで,Rviz上で4輪の移動ロボットが表示される.

### 4輪移動ロボットのコントロール
以下のコマンドを実行する.
端末1
```
$ roslaunch wheel_robot wheel_robot_control.launch
```
端末2
```
$ rqt
```
rqtを立ち上げた後,「Plugins」->「Robot Tools」->「Robot Steering」を選んでツールを立ち上げる.
縦軸は前後の速度指令,横軸は旋回角加速度の指令を行える.ツールバーを動かすことで,RvizとGazeboの両方で移動ロボットが動作することが確認できる.

### Gazebo上の移動の軌跡の保存
以下のコマンドを実行する.
端末1
```
$ roslaunch wheel_robot wheel_robot_control.launch
```
端末2
```
$ rosrun odom_listener odom_listener
```
端末3
```
$ rqt
```
上記の3つを実行後に,ロボットを所望する場所にコントロールして軌道を保存する.
所望する軌道通りに移動が終わったら,odom_listenerを実行した画面でctl+cでノードを終了する.
終了したウィンドウに「write success」が表示されれば保存できている.
rosrun odom_listener odom_listenerを実行したディレクトリ内に保存されている.

### 保存した軌跡の追従
以下のコマンドを実行することで追従を行えることを確認する.実行する前に,mapのディレクトリに適当なpgmファイル(ASCII形式)を保存し,waypointsのディレクトリに保存した軌跡を置いておく.
端末1
```
$ roslaunch wheel_robot wheel_robot_control.launch
```
端末2
```
$ roslaunch wheel_robot_nav wheel_robot_nav.launch
```
上記のコマンドを実行することで,waypointsに従って移動する.
Rvizの画面の「Add」を押し,「By topic」->「visualization_marker」->「MarkerArray」を選択・追加することで,waypointを表示できる.
次に,「Add」を押し,「By topic」->「current_goal」->「Pose」を選択・追加して,ゴールのポジションを表示する.
最後に,「Add」を押し,「By topic」->「NavfnROS」->「plan」->「Path」を選択・追加して,ゴールまでの計画経路を表示する.

### Gazebo上の4輪移動ロボットによる地図の作成
端末1
```
$ roslaunch wheel_robot wheel_robot.launch
```
端末2
```
$ roslaunch wheel_robot_gmapping wheel_robot_gmapping.launch
```
端末3
```
$ rqt
```
端末4
```
$ rosrun rviz rviz
```
端末1のコマンドを実行することで,gazeboの立ち上げを行っているため,gazebo内に障害物などを置いておく.
rqtを用いて移動ロボットを動かす.
Rvizの画面で,「Add」から「RobotModel」と「Map」を追加することで,地図ができていく様子を確認できる.
地図の保存には,保存したいディレクトリに移動し,以下のコマンドを実行する.
端末5
```
$ rosrun map_server map_saver -f test  (testは任意のファイル名)
```

### Gazebo上で構築した差動2輪の表示&Willow Garageの生成
以下のコマンドを実行する.
端末1
```
$ roslaunch diff_mobile_robot diff_mobile_robot.launch
```
端末2
```
$ rviz rviz
```
Rvizの「Fixed Frame」を「map」->「base_link」にし,「Add」から「RobotModel」を選択・追加する.
次に,以下のコマンドを実行する.
端末
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
```
コマンドを実行することで,差動2輪モデルとWillow Garageが表示されることが確認できる.
また,rqtを用いて移動ロボットがコントロールできるか確認する.

### Gazebo上の差動2輪移動ロボットによる地図の作成
下記コマンドを実行し,gmappingのインストールを行う.
端末
```
$ sudo apt-get install ros-kinetic-gmapping
```
以下のコマンドを実行する.
端末1
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
```
端末2
```
$ roslaunch diff_mobile_robot gmapping.launch
```
端末3
```
$ rqt
```
rqtを用いて差動2輪ロボットを動かすことで,地図が更新されていく.この様子は,RvizでmapをAddしておくことで確認することが確認できる.
地図が生成できた後に,地図を保存したいディレクトリに移動し,以下のコマンドで保存する.
端末4
```
$ rosrun map_server map_saver -f testmap  (testmapは任意のファイル名)
```

### AMCLでの差動2輪移動ロボットの自己位置推定
mapのディレクトリに先程作成した地図を保存する.
以下のコマンドを実行する.
端末1
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
```
端末2
```
$ roslaunch diff_mobile_robot amcl.launch
```
端末3
```
$ rviz rviz
```
端末4
```
$ rqt
```
rviz内で「Add」より「By display type」の「RobotModel」と「Map」,「By topic」の「particlecloud」->「PoseArray」を選択・追加することで,ロボットの移動に伴い,amclによる位置推定の様子が行われていることを確認できる.

### 差動2輪移動ロボットの保存した軌跡の追従
以下のコマンドを実行し,所望する軌道を保存する.
端末1
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
```
端末2
```
$ roslaunch diff_mobile_robot amcl.launch
```
端末3
```
$ rviz rviz
```
端末4
```
$ rqt
```
端末5
```
$ rosrun odom_listener odom_listener
```
次に,以下のコマンドを実行することで,保存した軌跡の追従を行えることを確認する.このコマンドを実行する前に,waypointsのディレクトリに保存した軌跡に置いておく.
端末1
```
$ roslaunch diff_mobile_robot diff_mobile_gazebo.launch
```
端末2
```
$ roslaunch diff_mobile_robot amcl.launch
```
端末3
```
$ rviz rviz
```
端末4
```
$  roslaunch diff_mobile_robot diff_mobile_robot_nav.launch
```
