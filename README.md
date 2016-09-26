<?php
defined('BASEPATH') OR exit('No direct script access allowed');

class Amazon_Backup extends CI_Controller {

	/**
	 * Index Page for this controller.
	 */
	public function __construct(){
		parent::__construct();
		error_reporting(E_ALL);
		ini_set('display_errors', 1);

		ini_set('max_execution_time', 600);
	 	ini_set('memory_limit','1024M');
                $this->load->library('s3');

		}


/* Function for creating the backup of Project and both the database */
	public function create_backup()
	{
	
		$this->zipData($_SERVER['DOCUMENT_ROOT'].'/', '/var/www/backupfile/backup-'.date('Y-m-d').'.zip'); //generate the full sitre backup
		$this->dbbackupfile(); // generate both database backup
		
		$file = '/var/www/backupfile/backup-'.date('Y-m-d').'.zip';
		
		/* Function for uploading the backup to the AWS Bucket */
		$database =  $this->db->database;
		$dbname = $database;
		$dbname2 = 'disco_master';
		
		$backup_file = $dbname .'-'. date("Y-m-d") . '.gz'; // get db filename 
		$backup_file2 = $dbname2 .'-'. date("Y-m-d") . '.gz'; // get db filename 
		
		$this->s3->putObjectFile($file, TCP_SALEFORCE, BACKUP_SALEFORCE.baseName('backup-'.date('Y-m-d').'.zip'), S3::ACL_PUBLIC_READ);
		$this->s3->putObjectFile($file, TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file), S3::ACL_PUBLIC_READ);
		$response = $this->s3->putObjectFile($file, TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file2), S3::ACL_PUBLIC_READ);
		
		/* check for the response and delete backup from the server*/
		if($response)
		{
			$this->deletedirContent();
		
		}
		
		/* Code for getting the previous 90th day day */
		$get_date = date('Y-m-d', strtotime(' -90 day'));
		$backup_file = $dbname .'-'. $get_date . '.gz'; // get db filename 
		$backup_file2 = $dbname2 .'-'. $get_date . '.gz'; // get db filename
		
		/* Function for deleting the object from bucket after 90 days */
		$this->s3->deleteObject(TCP_SALEFORCE, BACKUP_SALEFORCE.baseName('backup-'.$get_date.'.zip'));
		$this->s3->deleteObject(TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file));
		$this->s3->deleteObject(TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file2));

	}
	
	/** Function for deleting the backup zip from the server **/
	
	public function deletedirContent() {
	
	$dirname = '/var/www/backupfile/';
	array_map('unlink', glob("$dirname/*.*"));
	
	}
	
	/* Function for generreting the fullbackup of dev server files */
	
	public function zipData($source, $destination) {		
    if (extension_loaded('zip')) {
        if (file_exists($source)) {
        	//chmod($source, 0777);
            $zip = new ZipArchive();
            if ($zip->open($destination, ZIPARCHIVE::CREATE)) {
                $source = realpath($source);
                if (is_dir($source)) {
                    $iterator = new RecursiveDirectoryIterator($source);
                    // skip dot files while iterating 
                    $iterator->setFlags(RecursiveDirectoryIterator::SKIP_DOTS);
                    $files = new RecursiveIteratorIterator($iterator, RecursiveIteratorIterator::SELF_FIRST);
                    foreach ($files as $file) {
                        $file = realpath($file);
                        if (is_dir($file)) {
                            $zip->addEmptyDir(str_replace($source . '/', '', $file . '/'));
                        } else if (is_file($file)) {
                            $zip->addFromString(str_replace($source . '/', '', $file), file_get_contents($file));
                        }
                    }
                } else if (is_file($source)) {
                    $zip->addFromString(basename($source), file_get_contents($source));
                }
            }
            //chmod($source, 0755);
            //echo "<pre>";
            //print_r($zip);
            return $zip->close();
        }
    }
    return false;
} 	

	//database backup file created here
	public function dbbackupfile(){
	
	//get the database values
	$hostname = $this->db->hostname;
	$user =  $this->db->username;
	$pass =  $this->db->password;
	$database =  $this->db->database;
		
		//echo 'host->'.$hostname.'user->'.$user.'pass'.$pass.'database->'.$database;

	
	$dbname = $database;
	$dbname2 = 'disco_master';
	$dbhost = $hostname;
	$dbuser = $user;
	$dbpass = $pass;

	$backup_file = $dbname .'-'. date("Y-m-d") . '.gz';
	$backup_file2 = $dbname2 .'-'. date("Y-m-d") . '.gz';
	$command = "mysqldump --opt -h $dbhost -u $dbuser -p $dbpass ". "credit_dev | gzip > /var/www/backupfile/$backup_file";
	$command2 = "mysqldump --opt -h $dbhost -u $dbuser -p $dbpass ". "disco_master | gzip > /var/www/backupfile/$backup_file2";

	system($command);
	system($command2);
}

//database backup file created end here

/* Function for uploading the created backup files to the server */

public static function inputFile($file, $md5sum = true)
{
    if (!file_exists($file) || !is_file($file) || !is_readable($file))
    {
        trigger_error('S3::inputFile(): Unable to open input file: ' . $file, E_USER_WARNING);
        return false;
    }
    return array('file' => $file, 'size' => filesize($file),
        'md5sum' => $md5sum !== false ? (is_string($md5sum) ? $md5sum :
                        base64_encode(md5_file($file, true))) : '');
}

function mailfun(){
	echo 'ail'. mail('dilip@itexpertsindya.com','hello', 'asdfasdfasdf');
}



}
