
    
        function create_backup() {
		$this->zipData($_SERVER['DOCUMENT_ROOT'].'/', '/var/www/backupfile/backup-'.date('Y-m-d').'.zip'); //generate the full sitre backup
		$this->dbbackupfile(); // generate database backup
		
		$file = '/var/www/backupfile/backup-'.date('Y-m-d').'.zip'; // path of the zip file
		
		/* Function for uploading the backup to the AWS Bucket */
		
		$dbname = 'databasename';
		
		$backup_file = $dbname .'-'. date("Y-m-d") . '.sql'; // get db filename 
		$back_1 = '/var/www/backupfile/credit_dev-'.date('Y-m-d').'.sql'; //path of the databse file
                
                //$this->s3->putObjectFile(pathoffolder, bucketname, foldernamein_bucketfolder.baseName('backup-'.date('Y-m-d').'.zip'), S3::ACL_PUBLIC_READ);
		
                $this->s3->putObjectFile($file, TCP_SALEFORCE, BACKUP_SALEFORCE.baseName('backup-'.date('Y-m-d').'.zip'), S3::ACL_PUBLIC_READ);
		$response= $this->s3->putObjectFile($back_1, TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file), S3::ACL_PUBLIC_READ);
		
		/* check for the response and delete backup from the server*/
		if($response)
		{
                    $this->deletedirContent();
		}
		
		/* Code for getting the previous 90th day day */
		$get_date = date('Y-m-d', strtotime(' -90 day'));
		$backup_file = $dbname .'-'. $get_date . '.sql'; // get db filename 
		
		/* Function for deleting the object from bucket after 90 days */
                
              //  $this->s3->deleteObject(bucketname, foldernamein_bucketfolder.baseName('backup-'.$get_date.'.zip'));
		$this->s3->deleteObject(TCP_SALEFORCE, BACKUP_SALEFORCE.baseName('backup-'.$get_date.'.zip')); // delete folder 
		$this->s3->deleteObject(TCP_SALEFORCE, BACKUP_SALEFORCE.baseName($backup_file)); // delete database
	}
        
        /** Function for deleting the backup zip from the server **/
	function deletedirContent() {
            $dirname = '/var/www/backupfile/';
            array_map('unlink', glob("$dirname/*.*"));
	}
