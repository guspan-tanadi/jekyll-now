---
layout: post
title: CRUD MySQL PHP Encode ID of Data
date: 2018-5-23
---
The next of [other post]({% post_url 2018-5-17-mysql-php-ubuntu %}).  
Of Inspect Element, user can manipulate the id of a data.
Which make wrong action to the data.
By the use of base64 encode then decode it when need to show the real id.
The id of each data, client not know.
Some changes are:

index.php

{% highlight php %}
<style>
p {
background-color: #9d3
}
span {
float: right
}
</style>

<?php
include "config/hub.php";
$q = $hub->query("select kd, content from notes");

foreach($q as $r){
$mtime = sha1( microtime() );
$encd = base64_encode( $r['kd']. "." .$mtime );
?>

<p class="datum" data-kd="<?php echo $encd;?>">
	<span>
		<a href="op/?act=rem&kd=<?php echo $encd;?>">
		Delete
		</a>
	</span>
	<a href="?kd=<?php echo $encd;?>">
		<?php echo nl2br( $r['content'] ); ?>
	</a>
</p>

<?php
}

$kd = $_GET['kd'];

if($kd) {

$parse = explode(".", base64_decode($kd) );

$q = $hub->prepare("select kd, content from notes where kd = ?");
$q->execute( array( $parse[0] ) );

	if($q){
	$d = $q->fetch();
	}

}

if(!$d) {
$fe = "add";

} else if($d) {
$fe = "upd&kd=" . $kd;
}

?>

<form method="POST" action="<?php echo 'op/?act=' . $fe;?>">

<textarea rows="9" name="text">
<?php
if($d) {
echo $d['content'];
}
?>
</textarea>

<input type="submit">
</form>
{% endhighlight %}

op/add.php not in used. Instead, it would be
op/index.php

{% highlight php %}
<?php
$act = $_GET['act'];
$text = $_POST['text'];
$ctext = trim( $text );

$kd = $_GET['kd'];
$decd = explode(".", base64_decode($kd) );

include "../config/hub.php";

if($act == 'add'){

$q = $hub->prepare("insert into notes (content) values (?)");
$q->execute( array( $ctext ) );

} else if($act == 'upd'){

$q = $hub->prepare("update notes set content = ? where kd = ?");
$q->execute( array( $ctext, $decd[0] ) );

} else if($act == 'rem'){

$q = $hub->prepare("delete from notes where kd = ?");
$q->execute( array( $decd[0] ) );

}
	if($q){
	header("Location:/opnote/");
	}
?>
{% endhighlight %}
