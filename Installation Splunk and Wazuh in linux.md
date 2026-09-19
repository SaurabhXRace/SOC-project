# Splunk installation in linux
```
wget -O splunkforwarder.deb "https://download.splunk.com/products/universalforwarder/releases/9.4.2/linux/splunkforwarder-9.4.2-e9664af3d956-linux-amd64.deb" 
```

```
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

```
sudo dpkg -i splunkforwarder.deb
```

```
sudo /opt/splunkforwarder/bin/splunk start --accept-license
```

```
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

```
sudo /opt/splunkforwarder/bin/splunk stop
```

```    
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

```
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.0.150:9997
```

```
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.0.150:9997
```

```
sudo /opt/splunkforwarder/bin/splunk list forward-server
```
```
sudo /opt/splunkforwarder/bin/splunk restart
```

```
sudo /opt/splunkforwarder/bin/splunk list forward-server
```

