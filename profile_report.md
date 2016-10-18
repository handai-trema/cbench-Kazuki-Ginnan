学籍番号：33E16006
名前:銀杏一輝

#Cbenchのボトルネック

profileを用いてボトルネックをしらべたところ  


./bin/trema run ./lib/cbench.rb


によりcbench.rbを起動させたあと、


./bin/cbench --port 6653 --switches 1 --loops 5 --ms-per-test 10000 --delay 1000 --throughput



を実行した場合でボトルネックを調べた  

 %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 84.68    48.64     48.64        1 48640.00 48640.00  Thread#join
 84.68    97.28     48.64        1 48640.00 48640.00  IO.select
 84.68   145.92     48.64        2 24320.00 24320.00  TCPServer#accept
 84.66   194.55     48.63      102   476.76   476.76  Kernel#sleep
  3.01   196.28      1.73    38879     0.04     0.21  BinData::Struct#instantiate_obj_at
  2.58   197.76      1.48    34585     0.04     0.05  Kernel#define_singleton_method
  2.19   199.02      1.26    23276     0.05     0.98  BinData::Struct#snapshot
  2.14   200.25      1.23    24131     0.05     0.25  BinData::Struct#find_obj_for_name
  2.11   201.46      1.21    24067     0.05     2.64  Array#each
  2.09   202.66      1.20    21031     0.06     0.08  Kernel#dup
  1.98   203.80      1.14    70135     0.02     0.09  BasicObject#!=
  1.69   204.77      0.97    24131     0.04     0.06  BinData::Struct#base_field_name
  1.62   205.70      0.93    61063     0.02     0.02  BinData::Base#has_parameter?
  1.62   206.63      0.93    21785     0.04     0.12  BinData::BasePrimitive#method_missing
  1.51   207.50      0.87    35137     0.02     0.07  BinData::Struct#include_obj?
  1.50   208.36      0.86    56171     0.02     0.12  BinData::BasePrimitive#_value
  1.46   209.20      0.84    33181     0.03     0.03  BinData::SanitizedField#name_as_sym


スレッドやカーネルなどTCPの部分などで時間がかかっていることがわかる。  

Thread#joinはスレッドの実行が終了するまで、カレントスレッドを停止する。  
IO.selectは入出力待ちの処理に時間がかかっていることがわかる。  
Kernel#sleepはプログラムが停止している時間が長いことが原因に挙げられる。  
TCPServer#acceptは接続要求を受け続けている状態である。  

テキストを参考にすると、Tremaはシングルスレッドで動作するので、同時に1つpacket_inハンドラしか実行できないとある。  
そのため、Packet_inが同時に発生したとき、順番に処理していることになり、実行に時間がかかってしまう。  


　　





  



