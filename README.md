# Cisco-Catalyst-9000-platform-StackWise-Calisma-Notlarim
# Cisco Catalyst 9000 platform StackWise
Catalyst 9000 serisi iki switchin tek bir switch gibi davranmasına izin veren bir teknolojidir. Aynı konfigürasyonlara, aynı iletim durumlarına sahiptir. Bu teknoloji, yüksek kullanılabilirlik, ölçeklenebilirlik, yönetim ve bakım dahil olmak üzere ağ tasarımının tüm alanlarında geliştirmelere olanak tanır.

İki fiziksel switchin tek bir mantıksal switche sanallaştırılması, ağ topolojisinin tasarımını temelden değiştirir. En önemli değişiklerden biri bu teknoloji sayesinde loop (döngü) içermeyen bir topoloji sağlanır. Çünkü iki switch tek bir switch gibi davranır, böylelikle iki düğüm gibi değil tek bir düğüm gibi gözükür ve loop oluşmaz.
Ek olarak, StackWise Virtual , uygulamaların yanıt süresini geliştirmek için artırılmış bant genişliğiyle kesintisiz iletişim sağlayan Stateful Switch Over (SSO) , Non-Stop Forwarding (NSF) ve Multi-chassis EtherChannel (MEC) gibi Cisco yeniliklerini içerir.

StacWise Virtual’ın temel faydalarını şöyle sıralayabiliriz:

•	Loop içeren bir topoloji riskini azaltmak,

•	Erişim (Access) katmandan artan bant genişliği sayesinde mevcut yatırımlardan daha iyi bir geri dönüş sağlamak,

•	Konfigürasyon hatalarını azaltmak,

•	Hot Standby Routing Protocol (HSRP), GLBP ve VRRP gibi First Hop Redundancy Protokollerin (FHRP) ortadan kaldırmak,

•	Tek bir noktadan yönetimi sağlamak,

•	Daha az operasyonel arıza noktası sağlamaktır.

Bir StackWise Virtual domain’de bir switch “active” diğer switch “standby” olarak belirlenir. Bütün control plane işlemleri active olarak belirlenen switch tarafından yönetilir. Bunlar ;
•	Simple Network Management Protocol (SNMP), Telnet, Secure Shell (SSH) Protocol

•	Layer 2 protokolleri (Bridge Protocol Data Units [BPDUs], Protocol Data Units [PDUs], Link Aggregation Control Protocol [LACP], vb. )

•	Layer 3 protokolleri (routing protocols, vb.)

Active olarak belirlenen switchin başına bir şey gelmesi durumunda beklemede olan standby switch hemen active rolünü üstlenir. Bu durumları gözlemlemek için aşağıdaki komutu kullanabiliriz:

        Stackwise-Virtual#show redundancy
        
##### _StackWise Sanal MAC adresleri_

StackWise Virtual’da active duruma geçen switchin bütün Layer 3 interfacelerine bir MAC adresi atanır ve bu MAC adresleri, Cisco 9000 serisi switchlerin kendisinde bulunan EEPROM tarafından türetilir. Active olarak seçilen switch StackWise Virtual domain için de bir MAC adresi türetecektir. Ayrıca, active olan switchin başına bir durum gelse bile bu domain için belirlenen MAC adresi değişmeyecek ve kalıcı olacaktır. Bunun nedeni MAC adresini herhangi bir aksilik durumunda tekrar güncellememektedir. Bu durum varsayılan olarak böyledir, eğer bu süreyi değiştirmek istersek aşağıdaki komutu kullanabiliriz:

    SV-1 #stack-mac persistent timer 

Eğer bu süreleri ve MAC adreslerini görüntelemek istersek kullanacağımız komut aşağıdaki gibidir:

    SV-1#show switch
    
Eğer sistem yeniden başlatılırsa, bu sefer ilk olarak active belirlenen switch yani  MAC adresini türetmiş switch standby olup diğer switch active olursa sistem yeniden başlatılmadan önceki MAC adresi ve sonrasında türetilen MAC adresi farklılık gösterecektir. Çünkü bu sefer diğer switch sistem açılırken active olduğu için MAC adresini o üretecektir. Bu çoğu ortam için sorun teşkil etmez, çünkü sistem başlatıldığında MAC adreslerinin reklamını yapan ARP ( Address Resolution Protocol) frameleri gönderilir. 

Sonuç olarak, active olarak belirlenen switch StackWise Virtual için sanal bir MAC adresi türetecektir. Active olan switchin herhangi bir nedenden dolayı hata vermesiyle standby durumunda olan switch active switch olacaktır. Ama belirlenen sanal MAC adresi değişmeyecektir. 
1.Sistem yeniden başlatılırsa ve ilk başta belirlenen active switch hala düzelmediyse o standby durumunda kalacaktır, bu yüzden StackWise Virtual için MAC adres türetemeyecektir. Bu durumda sonradan active olan switch MAC adresini belirleyecektir. 
2.Eğer sistem yeniden başlatılmazsa ve eğer sonradan active olan switchin MAC adres türetmesini istersek aşağıdaki komutu çalıştırabiliriz:

    SV-1#stack-mac update force
    
##### _StackWise Virtual Konfigürasyonu_

        SV-1#conf t
        SV-1(config)#stackwise-virtual
        SV-1(config-stackwise-virtual)#domain 100

        SV-1(config)#interface range fortyGigabitEthernet 1/0/23-24
        SV-1(config-if-range)#stackwise-virtual link 1

        SV-1(config)#interface fortyGigabitEthernet 1/0/12
        SV-1(config-if)#stackwise-virtual dual-active-detection
        SV-1(config-if)#end

        SV-1#wr mem
        Building configuration... [OK]
        SV-1#reload
        Reload command is being issued on Active unit, this will reload the whole stack
        Proceed with reload? [confirm]
----------------------------------------------
        SV-2#conf t
        SV-2(config)#stackwise-virtual
        SV-2(config-stackwise-virtual)#domain 100

        SV-2(config)#interface range fortyGigabitEthernet 1/0/23-24
        SV-2(config-if-range)#stackwise-virtual link 1

        SV-2(config)#interface fortyGigabitEthernet 1/0/12
        SV-2(config-if)#stackwise-virtual dual-active-detection
        SV-2(config-if)#end

        SV-2#wr mem
        Building configuration... [OK]
        SV-2#reload
        Reload command is being issued on Active unit, this will reload the whole stack
        Proceed with reload? [confirm]

Yukarıdaki konfigürasyonlarda, ilk başta stackwise-virtual diyerek bu teknolojiyi cihazlarda aktif edildi. Sonra, StackWise Virtual için birer domain oluşturuldu. Daha sonra, hangi interfaceler arasında StackWise Virtual link kullanılacağını o interfaceler altında aktif edildi. Bu portlar otomatik olarak port channel formunda olur. Son olarak da bu iki switch kendi arasında konuşabilsin diye birer interface seçildi ve o interfaceler altında dual-active-detection diyerek aktif duruma getirildi. Bütün bunları konfigüre ettikten sonra sistemi kaydedip yeniden başlatılmazsa StackWise Virtual teknolojisi çalışmayacaktır. Bu yüzden en sonunda reboot komutu ile sistemi yeniden başlatıldı. Ve artık fiziksel olarak iki tane olan Cisco Catalyst 9000 serisi switchler mantıksal olarak tek bir switch haline gelmiş olur.
