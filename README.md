# My-data
<?php

$dbhost = 'localhost';  // your server name
$dbuser = '****';            // your user name
$dbpass = '****';          // your passward
$conn = mysqli_connect($dbhost, $dbuser, $dbpass);
if(! $conn )
{
  die('connection fail: ' . mysqli_error($conn));
}
echo 'connection sucess<br />';
mysqli_query($conn , "set names utf8mb4");
   
$hashtag=$argv[1];
$baseUrl = 'https://www.instagram.com/explore/tags/'. $hashtag .'/?__a=1';
$url = $baseUrl;
print_r($url);


while(1) {
    $json = json_decode(file_get_contents($url));
    #print_r($json);
    
    #print_r($json->graphql->hashtag->edge_hashtag_to_media->edges);
    if (isset($json->graphql->hashtag->edge_hashtag_to_media->edges)){

    print_r("###############");
    print_r($url);
    #print_r($json->tag->media->nodes);

    foreach ($json->graphql->hashtag->edge_hashtag_to_media->edges as $node_each){
        $post_id=$node_each->node->id;
        $post_time=gmdate('Y-m-d H:i:s',$node_each->node->taken_at_timestamp);
        $display_src=$node_each->node->display_url;
        $owner_id=$node_each->node->owner->id;
        $no_of_comments=$node_each->node->edge_media_to_comment->count;
        $no_of_like=$node_each->node->edge_liked_by->count;
        
    if(isset($node_each->node->edge_media_to_caption->edges[0]->node->text)){ $caption=(mysql_escape_string(str_replace(PHP_EOL,'',$node_each->node->edge_media_to_caption->edges[0]->node->text)));}
        else{ $caption='';}

    
    $sql = "REPLACE INTO hashtag_tbl ".
        "(post_id,post_time,URL,user_id,no_of_comments,no_of_like,name_tag,caption) ".
        "VALUES ".
        "($post_id,'$post_time','$display_src',$owner_id,$no_of_comments,$no_of_like,'$hashtag','$caption')";
    
    mysqli_select_db( $conn, 'Instagram' );
    $retval = mysqli_query( $conn, $sql );
    if(! $retval )
    {
      die('cannot insert in database: ' . mysqli_error($conn));
    }
 

 }

    if($json->graphql->hashtag->edge_hashtag_to_media->page_info->end_cursor == null) break;
    if($json == null) break;
    $url = $baseUrl.'&max_id='.$json->graphql->hashtag->edge_hashtag_to_media->page_info->end_cursor;
    
}


}
mysqli_close($conn);

?>
