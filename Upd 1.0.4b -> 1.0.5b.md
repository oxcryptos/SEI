 # 1.0.5b 30/06/22
 
 
 on block 1091248 https://sei.explorers.guru/block/1091248 chain must be upgraded to v 1.0.5b
 
 
	sudo systemctl stop seid && cd sei-chain
	git pull
	git checkout 1.0.5beta
	make install

	seid version --long | head
    
>  version 1.0.5beta 
> 
>  commit: 1ffb0829747f1ac62419ef610025a6ef52659186

	sudo systemctl restart seid && journalctl -u seid -f -o cat
