1. [URL Re-Writing (Mangling)](http://wiki.squid-cache.org/Features/AddonHelpers#Helper_protocols)
2. [url_rewrite_program](http://www.squid-cache.org/Doc/config/url_rewrite_program/)


__squid.conf__
```
# SQUID ANEH yg dipake 3.1.20 bisa pake juga squid 2.7 sesuaikan saja
# Do not modify '/var/ipfire/proxy/squid.conf' directly since any changes
# you make will be overwritten whenever you resave proxy settings using the
# web interface!
#
# Instead, modify the file '/var/ipfire/proxy/advanced/acls/include.acl' and
# then restart the proxy service using the web interface. Changes made to the
# 'include.acl' file will propagate to the 'squid.conf' file at that time.

shutdown_lifetime 5 seconds
icp_port 0

http_port 192.168.1.212:3128 transparent

cache_effective_user squid
cache_effective_group squid
umask 022

pid_filename /var/run/squid.pid

cache_mem 8 MB
error_directory /usr/lib/squid/errors/en

memory_replacement_policy heap GDSF
cache_replacement_policy heap LFUDA

access_log /var/log/squid/access.log
cache_log /var/log/squid/cache.log
cache_store_log none

log_mime_hdrs off
forwarded_for off
via off

acl within_timeframe time MTWHFAS 00:00-24:00

#acl all src all
acl localhost src 127.0.0.1/32
acl SSL_ports port 443 # https
acl SSL_ports port 563 # snews
acl Safe_ports port 80 # http
acl Safe_ports port 21 # ftp
acl Safe_ports port 443 # https
acl Safe_ports port 563 # snews
acl Safe_ports port 70 # gopher
acl Safe_ports port 210 # wais
acl Safe_ports port 1025-65535 # unregistered ports
acl Safe_ports port 280 # http-mgmt
acl Safe_ports port 488 # gss-http
acl Safe_ports port 591 # filemaker
acl Safe_ports port 777 # multiling http
acl Safe_ports port 3128 # Squids port (for icons)

acl IPFire_http  port 81
acl IPFire_https port 444
acl IPFire_ips              dst 192.168.1.212
acl IPFire_networks         src "/var/ipfire/proxy/advanced/acls/src_subnets.acl"
acl IPFire_servers          dst "/var/ipfire/proxy/advanced/acls/src_subnets.acl"
acl IPFire_green_network    src 192.168.1.0/24
acl IPFire_green_servers    dst 192.168.1.0/24
acl CONNECT method CONNECT

#Access to squid:
#local machine, no restriction
http_access allow         localhost

#GUI admin if local machine connects
http_access allow         IPFire_ips IPFire_networks IPFire_http
http_access allow CONNECT IPFire_ips IPFire_networks IPFire_https

#Deny not web services
http_access deny          !Safe_ports
http_access deny  CONNECT !SSL_ports

#Set custom configured ACLs
http_access allow IPFire_networks within_timeframe
http_access deny  all

#Strip HTTP Header
request_header_access X-Forwarded-For deny all
reply_header_access X-Forwarded-For deny all
request_header_access Via deny all
reply_header_access Via deny all

cache deny all

acl semua urlpath_regex .*?
acl kabeh urlpath_regex \.*?

url_rewrite_access allow kabeh
url_rewrite_access allow semua
url_rewrite_program /etc/squid/jalankan.pl
url_rewrite_children 20 startup=10 idle=5 concurrency=30




request_body_max_size 0 KB
visible_hostname ipfire.localdomain


max_filedescriptors 4096
```


__jalankan.pl__

```perl
#!/usr/bin/perl
#Coded and Tested By Keblux
#Modified and Tested By Me (Ces Pun)
#silahkan dibongkar2/dijual/dibuang/ tanpa menghilangkan kredit


$|=1;
while (<>) {
$input=$_;
@tmp=split(/ /,$input);
chomp(@tmp);
$url = $tmp[0];

if ($url =~ m/^http\:\/\/.*youtube.*videoplayback.*id.*/)
{
    @itag = m/[&?](itag=[0-9]*)/;
    @id = m/[&?](id=[a-zA-Z0-9\-\_]*)/;
    @range = m/[&?](range=[^\&\s]*)/;
    @yutub = "youtube-cespun-@id@itag@range";
    $url_hasil = &prosesscurl($url,@yutub,"youtube");
    print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/speedtest\/(upload.php|(.*[.](jpg|txt|png|ico|js)))\?(.*)/)
{
  $url_hasil = &prosesscurl($url,$2,"speedtest");
  print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/(.*[.](jp(e?g|e|2)|gif|png|tiff?|bmp|tga|svg|ico|swf|crx|js))/)
{
  $ces = $1 . $2;
  (my $cespun = $ces) =~ s/[\$#@~!&*()\[\];,:?^ `\\\/]+//g;
  $url_hasil = &prosesscurl($url,$cespun,"image");
  print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/(.*[.](3gp|mp(3|4)|flv|(m|f)1v|(m|f)4v|on2|fid|aac|asf|flac|mpc|nsv|og(g|m|a)|avi|mov|wm(a|v)|mp(e?g|a|e|v)|mk(a|v)))/)
{
  $ces = $1 . $2;
  (my $cespun = $ces) =~ s/[\$#@~!&*()\[\];,:?^ `\\\/]+//g;
  $url_hasil = &prosesscurl($url,$cespun,"audiovideo");
  print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/(.*[.](exe|ms(i|u|p)|cab|bin|mar|xpi|psf))/)
{
  $ces = $1 . $2;
  (my $cespun = $ces) =~ s/[\$#@~!&*()\[\];,:?^ `\\\/]+//g;
  $url_hasil = &prosesscurl($url,$cespun,"aplikasi");
  print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/(.*[.](z(ip|[0-9]{2})|r(ar|[0-9]{2})|7z|bz2|gz|tar|rpm|deb|xz|webm|iop|ini|amf|ts1|apk|pak))/)
{
  $ces = $1 . $2;
  (my $cespun = $ces) =~ s/[\$#@~!&*()\[\];,:?^ `\\\/]+//g;
  $url_hasil = &prosesscurl($url,$cespun,"compress");
  print $url_hasil ."\n";
} elsif ($url =~ m/^http:\/\/(.*)\/(.*[.](txt|pdf|xls|doc|ppt|rtf))/)
{
  $ces = $1 . $2;
  (my $cespun = $ces) =~ s/[\$#@~!&*()\[\];,:?^ `\\\/]+//g;
  $url_hasil = &prosesscurl($url,$cespun,"document");
  print $url_hasil ."\n";
} else {
   print $url. "\n";
}
}

#sub proces download dengan curl
sub prosesscurl
{
        my $url_prosess=$_[0];
        my $file=$_[1];
        my $myFolder=$_[2];

        #sesuaikan path /srv/web/ipfire/html/data/ dengan document root webserver anda!!PENTING!!
        if(-e "/srv/web/ipfire/html/data/$myFolder/$file") {
        $url_hasil="http://192.168.1.212:81/data/$myFolder/$file";
        } else {
                $url_hasil=$url_prosess;
                #bugs fixed delay diclient waktu streaming di pecah ke function download T_T ngak ngaruh
                &downloadcurl($url_prosess,$file,$myFolder);
        }

        #kirim hasil prosess ke atas
        return $url_hasil;

}
sub downloadcurl
{
        my $url_prosess='"' . $_[0] . '"' . " > /dev/null 2>&1 &";
        my $file=$_[1];
        my $myFolder=$_[2];
        #karena response yg didapat adalah 206 partial content, wget tidak dapat mendownload file terpaksa pake fetch, kalo belum ada install dulu!!PENTING!!
        system("curl -s -o /srv/web/ipfire/html/data/$myFolder/$file $url_prosess");
        #rubah permission agar dapat dibaca client, kambali rubah pathnya dengan document root webserver anda
        chmod(0644, "/srv/web/ipfire/html/data/$myFolder/$file");
}
```
