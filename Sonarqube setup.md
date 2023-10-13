# Set up for nexus.
## Launch an EC2 Instance:
- Log in to your AWS Management Console.
- Launch a new EC2 instance. Ensure that you select an appropriate instance type and security groups.
- Now go to advance details and add the script in the user data
  
      #!/bin/bash
      yum install java-1.8.0-openjdk.x86_64 wget -y   
      mkdir -p /opt/nexus/   
      mkdir -p /tmp/nexus/                           
      cd /tmp/nexus/
      NEXUSURL="https://download.sonatype.com/nexus/3/latest-unix.tar.gz"
      wget $NEXUSURL -O nexus.tar.gz
      sleep 10
      EXTOUT=`tar xzvf nexus.tar.gz`
      NEXUSDIR=`echo $EXTOUT | cut -d '/' -f1`
      sleep 5
      rm -rf /tmp/nexus/nexus.tar.gz
      cp -r /tmp/nexus/* /opt/nexus/
      sleep 5
      useradd nexus
      chown -R nexus.nexus /opt/nexus 
      cat <<EOT>> /etc/systemd/system/nexus.service
      [Unit]                                                                          
      Description=nexus service                                                       
      After=network.target                                                            
                                                                        
      [Service]                                                                       
      Type=forking                                                                    
      LimitNOFILE=65536                                                               
      ExecStart=/opt/nexus/$NEXUSDIR/bin/nexus start                                  
      ExecStop=/opt/nexus/$NEXUSDIR/bin/nexus stop                                    
      User=nexus                                                                      
      Restart=on-abort                                                                
                                                                        
      [Install]                                                                       
      WantedBy=multi-user.target                                                      
      
      EOT
      
      echo 'run_as_user="nexus"' > /opt/nexus/$NEXUSDIR/bin/nexus.rc
      systemctl daemon-reload
      systemctl start nexus
      systemctl enable nexus
## Access Nexus Web Interface:

 - Once Nexus is running, you can access its web interface by opening a web browser and navigating to http://your_ec2_instance_ip:8081.
 - You should be able to log in with the default credentials (admin/admin123).
