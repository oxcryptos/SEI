	
  
    sudo systemctl stop seid && \
    sudo rm -rf $HOME/sei-chain && \
    git clone https://github.com/sei-protocol/sei-chain.git && \
    cd sei-chain && \
    git checkout 1.0.4beta && \
    make install && \
    seid version --long | head
    
> version 1.0.4beta 
> 
>  commit: 09b267bbc7d32d3f61ab1b186f59f4bc5fea8970

	  sudo systemctl restart seid && journalctl -u seid -f -o cat
