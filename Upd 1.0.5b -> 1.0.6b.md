 # 1.0.6b 04/07/22
 
 
 on block 1217302 https://sei.explorers.guru/block/1091248 chain must be upgraded to v 1.0.6b
 
 
	sudo systemctl stop seid && cd sei-chain
	git pull
	git checkout 1.0.6beta
	make install

	seid version --long | head
	
    
>  version 1.0.6beta 
> 
>  commit: e3958ff9cc3fa00a12b0c32cf55b635baa0d49bd

	sudo systemctl restart seid && journalctl -u seid -f -o cat
	
