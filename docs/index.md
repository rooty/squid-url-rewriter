1. [URL Re-Writing (Mangling)](http://wiki.squid-cache.org/Features/AddonHelpers#Helper_protocols)
2. [url_rewrite_program](http://www.squid-cache.org/Doc/config/url_rewrite_program/)

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


}


  else {
   print $url. "\n";
}
}
#}
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
