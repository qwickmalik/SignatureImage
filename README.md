# SignatureImage


## Step 1 :

Write below code in `app/Config/bootstrap.php`

```php
CakePlugin::load('SignatureImage');
```


## Step 2 :

Write below code in your contoller in which you want to create signatute to image

```php
public $components = array('SignatureImage.SignatureToImage');

public $helpers = array('SignatureImage.SignatureToImage');
```


## Step 3 :

Put below code in your `ctp` file. It creates a singaturebox where you can draw you signature.
**PS: This is where this fork differs from the original plubin by @chetanspeed511987**

```php
<?php echo $this->SignatureToImage->accept($id='mySignature'); ?>
```


## Step 4 :

Put below code in your particular action of controller. It creates a new signature image in your folder from json. 

```php
$json = $this->request->data['Signature']['signature']; // From Signature Pad
$img = $this->SignatureToImage->signJsonToImage($json);

$sign_image = WWW_ROOT.'img/signature'.rand().'.png';
$this->SignatureToImage->saveImage($img, $sign_image);
```

## Example :
**MySQL: **

```
-- Table structure for table `signatures`

DROP TABLE IF EXISTS `signatures`;
CREATE TABLE IF NOT EXISTS `signatures` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `signature` mediumtext DEFAULT NULL,
  `signature_path` mediumtext DEFAULT NULL,
  KEY `id` (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;
COMMIT;
```

**Model: ** The code below goes into app/Model/Signature.php
```
class Signature extends AppModel {

    var $name = "Signature";
    var $usesTable = "signatures";
     var $displayField = "signature";
       var $hasMany = array(
        
        );
    
}
```

**View :** Below code is the `addSignature.ctp` where we make a signaturebox.

```php
<div>
<?php 
	echo $this->Form->create('MyForm', array('class' => 'sigPad2', 'url' => array('controller' => 'MyController', 'action' => 'addSignature')));
	echo $this->SignatureToImage->accept($id='gee'); 
	if($latest_signature['Signature']['signature'] != null){
		echo $this->SignatureToImage->regenerate($latest_signature['Signature']['signature']); //this will regenerate the signature from json
	}
	if($latest_signature['Signature']['signature_path'] != null){
		echo $latest_signature['Signature']['signature_path']; //echo signature path
	}
	echo $this->Form->submit('Save'); 
	echo $this->Form->end(); 
        ?>
</div>
```

**Controller :** Below, we convert json to image and save the new signature image in define folder. The json object and path to the image file are saved to the signatures table

```php
public function addSignature() { 
        if ($this->request->is('post')) { 
            	$this->Signature->create(); 
//            	pr($this->request->data); exit; //wanna view the json? uncomment this line
            	
	    	$json = $this->request->data['Signature']['signature'] = $this->request->data['MyForm']['gee']; // From Signature Pad
		$img = $this->SignatureToImage->signJsonToImage($json);
		$now = date('Y-m-d-H_s');
		$sign_path = WWW_ROOT.'img/signatures/signature'.$now.'.png';
		$this->SignatureToImage->saveImage($img, $sign_path);
		$this->request->data['Signature']['signature_path'] = $sign_path;
		
            if ($this->Signature->save($this->request->data)) {
                $this->Session->setFlash(__('The signature has been saved'));
                return $this->redirect(array('controller' => 'MyController', 'action' => 'addSignature'));
		} else {
			$this->Session->setFlash(__('The signature could not be saved. Please, try again.'));
		}
	
	}
        $latest_signature = $this->Signature->find('first', array('order' => array('Signature.id' => 'desc')));
	$this->set('latest_signature', $latest_signature);
}
```

