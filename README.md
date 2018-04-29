# Ethereum Mining pool written in PHP
This is full source of Ethereum Mining Pool - http://ethereumpool.co It has been written mostly in php, code is not bad, but needs some work, it runs fine on one decent dedicated server if it's not serving more than ~500 workers, for more it needs to be rewritten and split between more servers. It should be considered alpha code. Code is messy, my intention at time was to swiftly create pool to save community from only one existing pool with more than 3/4 network hashrate.

<b>Disclaimer</b><br>
This is latest avaiable source code from original <b>ethereumpool.co</b> mining pool.<br>Domain has been sold to <b>eth.pp.ua</b>

# Requirements
Recommended <a href="https://github.com/facebook/hhvmHHVM" target="_blank">HHVM</a>, not mandatory, can be replaced with php-fpm<br>
<a href="https://mariadb.org" target="_blank">MariaDB server</a><br>
<a href="http://memcached.org" target="_blank">Memcached</a><br>
<a href="https://github.com/phpseclib/phpseclib/blob/master/phpseclib/Math/BigInteger.php" target="_blank">BigInteger for php (included in project)</a><br>
<a href="http://nginx.org" target="_blank">Nginx</a><br>
<a href="https://github.com/ethereum/pyethereum" target="_blank">Pyethereum</a><br>
<a href="https://github.com/ethereum/go-ethereum" target="_blank">Geth</a><br>

# Setup on Linux
Install all software mentioned above.<br>
Setup your mysql server and import database scheme <pre>misc>database_scheme.sql</pre>
Setup valid mysql connection details in config.php.
<br>
Setup nginx server blocks:
<pre>'mainpage' directory as public and if you need block /logs directory
'block_processing' locally
</pre>
Setup phpmyadmin with ssl, etc..<br>
Now move files <pre>nonce_fast.py and nonce.py</pre> from 'misc' directory to <pre>/root/pyethereum/ethereum/ (main directory of Pyethereum)</pre><br>
To maximize performance, tweak configurations of memcached,mariadb,nginx,hhvm and kernel but it's not necessary to start pool.<br>

# Start Pool
<pre>screen -dmS geth geth --rpcaddr 127.0.0.1 --rpcport 8983 --rpc --unlock Address</pre>

Now start background scripts:<br>
Get work from geth json and cache it with memcached
<pre>screen -dmS process_work php /var/www4/block_processing/process_work/index.php</pre>
<br>Block Processing - this script handle block splitting and proof of work verification
<pre>screen -dmS block_processing php /var/www4/block_processing/index.php</pre>
<br>This script updates data to calculate predicted mining rewards
<pre>screen -dmS update_calculator php /var/www4/block_processing/update_calculator/index.php</pre>
<br>Used to process internal statistics and save to database
<pre>screen -dmS stats php /var/www4/block_processing/stats/index.php</pre>

<br>
You can execute withdraws manually or add it as cron job
<pre>sudo php /var/www4/block_processing/withdraw/index.php</pre>

crontab -e
<pre>* */12 * * * sudo php /var/www4/block_processing/withdraw/index.php</pre>

This both scripts can be used to check if withdraws has been processed correctly or check if splited balance == real balance, it was mainly used while development process but it might be helpful.
<pre>
sudo php /home/www4/block_processing/withdraw_check/index.php
curl http://127.0.0.1:9846/check/</pre>

# Notes
withdraw_check and withdraw scripts saves logs in block_processing directory.<br>
If you would like to debug mining proxy (mainpage/index.php)
<pre>$logstate = true;</pre>
Don't use it on production, it's quite heavy.<br>

Make sure to setup valid permissions, allow php-fpm to run python script and make sure that directory permissions are correct to save logs.
Some paths may require changes.<br>
<br>
Setting up pool revenue address and fee<br>
<pre>block_processing/index.php</pre>make sure address exists as 'miner' in miners table.

<br><br>
You can easily access all background scripts by
<pre>
screen -x stats/update_calculator/block_processing/process_work/geth</pre>

# Contributing
If you want to contribute, fork and pull request or open issue.


# License
php,python and some js,css,html code is under The MIT License (MIT) including charting library Plotly.js <br>
Front end template is [paid](http://themes.3rdwavemedia.com/website-templates/responsive-bootstrap-theme-web-development-agencies-devstudio) (sorry, at first it wasn't meant to be open sourced thus i just picked paid template)
