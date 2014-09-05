##Building a REST API in Yii2.0
This is a simple wiki which is useful if you are trying to build a REST API in Yii2.0

Note:This example is based on a table user(id(PK AI),name,age,createdAt,updatedAt)

###1.Action index

      Request:
               URL: api/user/index
            method:GET 
	    params:
```javascript
	             {
	              "page":1,
	              "limit":5,
	              "sort":"id",
	              "order":false,
	              "filter":{}
	             }
```
           Note1: "page"=>is the current page number
           Note2: "limit"=>no.of records in a single page
           Note3: "sort"=>sort field(ie this can be id,name,age createdAt or updatedAt)
           Note4: "order"=>This can be true/false. true=>ascending order while false=>descending order
           Note5: filter=>is a json object to pass any filter elements. eg:{name:'abc',age:20} 
           
     ####Response:
```javascript
		     {
			  "status": 1,
			  "data": [
			      {
				  "id": 30,
				  "name": "john",
				  "age": 78,
				  "createdAt": "2014-09-05 01:53:31",
				  "updatedAt": "2014-09-05 01:53:51"
			      },
			      {
				  "id": 29,
				  "name": "ben",
				  "age": 23,
				  "createdAt": "2014-09-05 01:53:28",
				  "updatedAt": "2014-09-05 01:54:00"
			      },
			      {
				  "id": 28,
				  "name": "rahul",
				  "age": 72,
				  "createdAt": "2014-09-05 01:53:25",
				  "updatedAt": "2014-09-05 01:54:09"
			      },
			      {
				  "id": 27,
				  "name": "shafeeque",
				  "age": 76,
				  "createdAt": "2014-09-05 01:53:21",
				  "updatedAt": "2014-09-05 01:54:24"
			      },
			      {
				  "id": 26,
				  "name": "sirin",
				  "age": 73,
				  "createdAt": "2014-09-04 19:51:49",
				  "updatedAt": "2014-09-05 01:54:32"
			      }
			  ],
			  "totalItems": "8"
		      }
```
####Action Source code:

```php
public function actionIndex()
{

      $params=$_REQUEST;
      $filter=array();
    
      $page=1;  /* default page number */
      $limit=10; /* default limit */
	
	  
      if(isset($params['page']))
	$page=$params['page'];
	  
	  
      if(isset($params['limit']))
	  $limit=$params['limit'];
	  
	$offset=$limit*($page-1);
	
	
	/* Filter elements */
      if(isset($params['filter']))
	{
	$filter=(array)json_decode($params['filter']);
	}
	
	/* Sort field */
	$sort="";
	if(isset($params['sort']))
	{
	  $sort=$params['sort'];
	    if(isset($params['order']))
	    {  
		if($params['order']=="false")
		$sort.=" desc";
		else
		$sort.=" asc";
	    
	    }
	}
    
	  
	  
	    $models=User::find()
			->offset($offset)
			->limit($limit)
			
			->andFilterWhere(['like', 'id', $filter['id']])
			->andFilterWhere(['like', 'name', $filter['name']])
			->andFilterWhere(['like', 'age', $filter['age']])
			
			->orderBy($sort)
			->all();
	    
	    
	    $totalItems=User::find()
			    
			    ->andFilterWhere(['like', 'id', $filter['id']])
			    ->andFilterWhere(['like', 'name', $filter['name']])
			    ->andFilterWhere(['like', 'age', $filter['age']])
			    ->count();
		  
		  
      
      $data=array();
      foreach($models as $m)
      { 
	$data[]=array_filter($m->attributes);
      }     
      
      $this->setHeader(200);
    
      echo json_encode(array('status'=>1,'data'=>$data,'totalItems'=>$totalItems),JSON_PRETTY_PRINT);
  
}

/* Functions to set header with status code. eg: 200 OK ,400 Bad Request etc..*/	    
private function setHeader($status)
{
    
    $status_header = 'HTTP/1.1 ' . $status . ' ' . $this->_getStatusCodeMessage($status);
    $content_type="application/json; charset=utf-8";
  
    header($status_header);
    header('Content-type: ' . $content_type);
    header('X-Powered-By: ' . "Nintriva <nintriva.com>");
}
private function _getStatusCodeMessage($status)
{
    $codes = Array(
	200 => 'OK',
	400 => 'Bad Request',
	401 => 'Unauthorized',
	402 => 'Payment Required',
	403 => 'Forbidden',
	404 => 'Not Found',
	500 => 'Internal Server Error',
	501 => 'Not Implemented',
    );
    return (isset($codes[$status])) ? $codes[$status] : '';
}	   
```
###2.Action View
 
	       Request:
               URL: api/user/view/30
	    method:GET
	    
           Note1: "30"=>is the Pk of a record in the user table
           
             ####Response:
```javascript             

{
      "status": 1,
      "data": {
	  "id": 30,
	  "name": "john",
	  "age": 78,
	  "createdAt": "2014-09-05 01:53:31",
	  "updatedAt": "2014-09-05 01:53:51"
      }
}
```			  
####Action Source code:	

```php          
public function actionView($id)
{

  $model=$this->findModel($id);
  
  $this->setHeader(200);
  echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
    
} 
  /* function to find the requested record/model */
protected function findModel($id)
{
    if (($model = User::findOne($id)) !== null) {
	return $model;
    } else {
    
      $this->setHeader(400);
      echo json_encode(array('status'=>0,'error_code'=>400,'message'=>'Bad request'),JSON_PRETTY_PRINT);
      exit;
    }
}
###3.Action Create
 
	       Request:
	       
                URL:  api/user/create
	     method:POST
####params:
```javascript
	{
	  "name":"abc",
	  "age":20,
	}
```	
      
####Response:
```javascript
	  {
	      "status": 1,
	      "data": {
		  "id": 32,
		  "name": "abc",
		  "age": "20",
		  "createdAt": "2014-09-05 02:35:18",
		  "updatedAt": "2014-09-05 02:35:18"
	      }
	  }
```	 
####Action Source code:	
     
```php     
public function actionCreate()
{
  
    $params=$_REQUEST;
    
    $model = new User();
    $model->attributes=$params;
    
    
    if ($model->save()) {
    
	$this->setHeader(200);
	echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
      
    } 
    else
    {
	$this->setHeader(400);
	echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
    }

}
```

###4.Action Update
 
	       Request:
	       
                URL: api/user/update/32
                 
                  
                Note1:"32"=>id(PK) of the record we are going to update
                
                
	     method: POST
	     
####params:
```javascript
	  {
	    "name":"efg",
	    "age":25,
	  }
```		
	
####Response:
 ```javascript
	    {
		"status": 1,
		"data": {
		    "id": 32,
		    "name": "efg",
		    "age": "25",
		    "createdAt": "2014-09-05 02:35:18",
		    "updatedAt": "2014-09-05 02:45:55"
		}
	    }
 ```	 
####Action Source code:	
```php
                          
public function actionUpdate($id)
{
    $params=$_REQUEST;

    $model = $this->findModel($id);

    $model->attributes=$params;

    if ($model->save()) {
    
	$this->setHeader(200);
	echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
      
    } 
    else
    {
	$this->setHeader(400);
	echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
    }
    
}		
```

###5.Action Delete
 
	       Request:
	       
                URL: api/user/update/32
             method:DELETE 
                  
                Note1:"32"=>id(PK) of the record we are going to delete
         
	             
####Response:

```javascript
	  {
	      "status": 1,
	      "data": {
		  "id": 32,
		  "name": "efg",
		  "age": 20,
		  "createdAt": "2014-09-05 02:40:44",
		  "updatedAt": "2014-09-05 02:40:44"
	      }
	  }
```	  
####Action Source code:	
        
```php        
public function actionDelete($id)
{
    $model=$this->findModel($id);
    
    if($model->delete())
    { 
	$this->setHeader(200);
	echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
    
    }
    else
    {
      
	$this->setHeader(400);
	echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
    }

}	
```

###6.Action DeleteAll
  
    Used to delete multiple records at a time.
 
	       Request:
	       
                URL: api/user/deleteall
             method:POST
####params:
```javascript
{
  "ids":"[27,28]"
}
```
                  
                Note1:"ids"=>a list of id(Pk)'s to be deleted
         
	             
####Response:
```javascript
{
    "status": 1,
    "data": [
	{
	    "id": 27,
	    "name": "shafeeque",
	    "age": 76,
	    "createdAt": "2014-09-05 01:53:21",
	    "updatedAt": "2014-09-05 01:54:24"
	},
	{
	    "id": 28,
	    "name": "rahul",
	    "age": 72,
	    "createdAt": "2014-09-05 01:53:25",
	    "updatedAt": "2014-09-05 01:54:09"
	}
    ]
}
```
####Action Source code:	
```php                          
public function actionDeleteall()
{
    $ids=json_decode($_REQUEST['ids']);
  
    $data=array();
    
    foreach($ids as $id)
    {
      $model=$this->findModel($id);
      
      if($model->delete())
	$data[]=array_filter($model->attributes);
      else
      {
	$this->setHeader(400);
	echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
	return;
      }  
    }
    
    $this->setHeader(200);
    echo json_encode(array('status'=>1,'data'=>$data),JSON_PRETTY_PRINT);

}
php
			  
###7.Behaviour to fileter action methods

```php
public function behaviors()
{
    return [
	'verbs' => [
	    'class' => VerbFilter::className(),
	    'actions' => [
		'index'=>['get'],
		'view'=>['get'],
		'create'=>['post'],
		'update'=>['post'],
		'delete' => ['delete'],
		'deleteall'=>['post'],
	    ],
	  
	]
    ];
}

  /* This will execute before  any action */
public function beforeAction($event)
{
    $action = $event->id;
    if (isset($this->actions[$action])) {
	$verbs = $this->actions[$action];
    } elseif (isset($this->actions['*'])) {
	$verbs = $this->actions['*'];
    } else {
	return $event->isValid;
    }
    $verb = Yii::$app->getRequest()->getMethod();
  
  $allowed = array_map('strtoupper', $verbs);
  
  if (!in_array($verb, $allowed)) {
	
	$this->setHeader(400);
	echo json_encode(array('status'=>0,'error_code'=>400,'message'=>'Method not allowed'),JSON_PRETTY_PRINT);
	exit;
	
    }  
    
  return true;  
}
```

####Controller source code:
 
```php
	    namespace app\modules\api\controllers;

	    use Yii;
	    use app\models\User;
	    use yii\data\ActiveDataProvider;
	    use yii\web\Controller;
	    use yii\web\NotFoundHttpException;
	    use yii\filters\VerbFilter;

	    /**
	    * UserController implements the CRUD actions for User model.
	    */
	    class UserController extends Controller
	    {
	      
		public function behaviors()
		{
		    return [
			'verbs' => [
			    'class' => VerbFilter::className(),
			    'actions' => [
				'index'=>['get'],
				'view'=>['get'],
				'create'=>['post'],
				'update'=>['post'],
				'delete' => ['delete'],
				'deleteall'=>['post'],
			    ],
			  
			]
		    ];
		}
		
	      
		public function beforeAction($event)
		{
		    $action = $event->id;
		    if (isset($this->actions[$action])) {
			$verbs = $this->actions[$action];
		    } elseif (isset($this->actions['*'])) {
			$verbs = $this->actions['*'];
		    } else {
			return $event->isValid;
		    }
		    $verb = Yii::$app->getRequest()->getMethod();
		  
		  $allowed = array_map('strtoupper', $verbs);
		  
		  if (!in_array($verb, $allowed)) {
			
			$this->setHeader(400);
			echo json_encode(array('status'=>0,'error_code'=>400,'message'=>'Method not allowed'),JSON_PRETTY_PRINT);
			exit;
			
		    }  
		    
		  return true;  
		}
	      
		/**
		* Lists all User models.
		* @return mixed
		*/
		public function actionIndex()
		{
		
		      $params=$_REQUEST;
		      $filter=array();
		      $sort="";
		    
		      $page=1;
		      $limit=10;
			
		      if(isset($params['page']))
			$page=$params['page'];
			  
			  
		      if(isset($params['limit']))
			  $limit=$params['limit'];
			  
			$offset=$limit*($page-1);
			
			
			/* Filter elements */
		      if(isset($params['filter']))
			{
			$filter=(array)json_decode($params['filter']);
			}
		
			
			
			if(isset($params['sort']))
			{
			  $sort=$params['sort'];
			    if(isset($params['order']))
			    {  
				if($params['order']=="false")
				$sort.=" desc";
				else
				$sort.=" asc";
			    
			    }
			}
		    
			  
			  
			    $models=User::find()
					->offset($offset)
					->limit($limit)
					
					->andFilterWhere(['like', 'id', $filter['id']])
					->andFilterWhere(['like', 'name', $filter['name']])
					->andFilterWhere(['like', 'age', $filter['age']])
				      
				      
					->orderBy($sort)
					->all();
			    
			    
			    $totalItems=User::find()
					    ->andFilterWhere(['like', 'id', $filter['id']])
					    ->andFilterWhere(['like', 'name', $filter['name']])
					    ->andFilterWhere(['like', 'age', $filter['age']])
					    ->count();
				  
				  
		      
		      $data=array();
		      foreach($models as $m)
		      { 
			$data[]=array_filter($m->attributes);
		      }     
		      
		      $this->setHeader(200);
		    
		      echo json_encode(array('status'=>1,'data'=>$data,'totalItems'=>$totalItems),JSON_PRETTY_PRINT);
		  
		}
		  

		/**
		* Displays a single User model.
		* @param integer $id
		* @return mixed
		*/
		public function actionView($id)
		{
	      
		  $model=$this->findModel($id);
		  
		  $this->setHeader(200);
		  echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
		    
		}

		/**
		* Creates a new User model.
		* @return json
		*/
		public function actionCreate()
		{
		  
		    $params=$_REQUEST;
		    
		    $model = new User();
		    $model->attributes=$params;
		    
		    

		    if ($model->save()) {
		    
			$this->setHeader(200);
			echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
		      
		    } 
		    else
		    {
			$this->setHeader(400);
			echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
		    }
		
		}

		/**
		* Updates an existing User model.
		* @param integer $id
		* @return json
		*/
		public function actionUpdate($id)
		{
		    $params=$_REQUEST;
		
		    $model = $this->findModel($id);
		
		    $model->attributes=$params;

		    if ($model->save()) {
		    
			$this->setHeader(200);
			echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
		      
		    } 
		    else
		    {
			$this->setHeader(400);
			echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
		    }
		    
		}

		/**
		* Deletes an existing User model.
		* @param integer $id
		* @return json
		*/
		public function actionDelete($id)
		{
		    $model=$this->findModel($id);
		    
		    if($model->delete())
		    { 
			$this->setHeader(200);
			echo json_encode(array('status'=>1,'data'=>array_filter($model->attributes)),JSON_PRETTY_PRINT);
		    
		    }
		    else
		    {
		      
			$this->setHeader(400);
			echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
		    }

		}
		/**
		* Deletes an existing multiple User models at a time.
		* @return json
		*/
		public function actionDeleteall()
		{
		    $ids=json_decode($_REQUEST['ids']);
		  
		    $data=array();
		    
		    foreach($ids as $id)
		    {
		      $model=$this->findModel($id);
		      
		      if($model->delete())
			$data[]=array_filter($model->attributes);
		      else
		      {
			$this->setHeader(400);
			echo json_encode(array('status'=>0,'error_code'=>400,'errors'=>$model->errors),JSON_PRETTY_PRINT);
			return;
		      }  
		    }
		    
		    $this->setHeader(200);
		    echo json_encode(array('status'=>1,'data'=>$data),JSON_PRETTY_PRINT);

		}

		/**
		* Finds the User model based on its primary key value.
		* If the model is not found, a 404 HTTP exception will be thrown.
		* @param integer $id
		* @return User the loaded model
		*/
		protected function findModel($id)
		{
		    if (($model = User::findOne($id)) !== null) {
			return $model;
		    } else {
		    
		      $this->setHeader(400);
		      echo json_encode(array('status'=>0,'error_code'=>400,'message'=>'Bad request'),JSON_PRETTY_PRINT);
		      exit;
		    }
		}
		
		private function setHeader($status)
		  {
		      
		      $status_header = 'HTTP/1.1 ' . $status . ' ' . $this->_getStatusCodeMessage($status);
		      $content_type="application/json; charset=utf-8";
		    
		      header($status_header);
		      header('Content-type: ' . $content_type);
		      header('X-Powered-By: ' . "Nintriva <nintriva.com>");
		  }
		private function _getStatusCodeMessage($status)
		{
		    // these could be stored in a .ini file and loaded
		    // via parse_ini_file()... however, this will suffice
		    // for an example
		    $codes = Array(
			200 => 'OK',
			400 => 'Bad Request',
			401 => 'Unauthorized',
			402 => 'Payment Required',
			403 => 'Forbidden',
			404 => 'Not Found',
			500 => 'Internal Server Error',
			501 => 'Not Implemented',
		    );
		    return (isset($codes[$status])) ? $codes[$status] : '';
		}
	    }
```

	    
Happy coding..

Sirin

Nintriva

www.nintriva.com

	    
	
	      
		     