package
{
	import flash.errors.SQLError;
	
	/* Class to encapsulate DB access for Mood Maestro */
	public class MMDBAccess
	{
		import flash.events.SQLEvent;
		import flash.events.Event;
		import flash.events.SQLErrorEvent;
		import flash.data.SQLConnection;
		import flash.data.SQLStatement;
		import flash.data.SQLResult;
		import flash.filesystem.File;
		private static var mmDB:SQLConnection = new SQLConnection();  
		private static var mmDBA:SQLConnection = new SQLConnection();  
		private static var addMoodStmt:SQLStatement = new SQLStatement();    
		private static var selectMoodStmt:SQLStatement = new SQLStatement();
		private static var selectPrefStmt:SQLStatement = new SQLStatement();
		private static var result:SQLResult;
		
		public function MMDBAccess()
		{
		}
	
		public static function createDBTables():void {
			// set up db connection
			trace('initial DB and table creation');
			var dbFile:File = File.applicationStorageDirectory.resolvePath("MoodMaestro.db");  
			try {
				mmDB.open(dbFile);					// create syncronous connection for initial setup and prefs
				setupDataTable();					// setup the mood data table
				setupPrefsTable();					// set up the prefs table
				initDBStmts();						// initialize the sql statements for later use, this stuff is async, so we will return from here.
			}
			catch (error:SQLError) {
				trace("error in DB:", error.message, "details:", error.details);
			}
		}
		private static function initDBStmts():void {
			//Open the async connection and precreate select/update stmts
			createSyncStmts();
			var dbFile:File = File.applicationStorageDirectory.resolvePath("MoodMaestro.db");  
			mmDBA.addEventListener(SQLEvent.OPEN, createAsyncStmts);
			mmDBA.addEventListener(SQLErrorEvent.ERROR, errorHandler);
			mmDBA.openAsync(dbFile);			// create async connection for regular use
		}
		
		private static function errorHandler(errorEvent:SQLErrorEvent):void  
		{  
			trace("Details:", errorEvent.text);  
			//btn.enabled = false;				// disable the post button
		} 
			
		private static function setupDataTable():void {
			// create the db tables if necessary
			var createStmt:SQLStatement = new SQLStatement();  
			createStmt.sqlConnection = mmDB;  
			var sql:String =   
			"CREATE TABLE IF NOT EXISTS moods (" +   
			" id INTEGER PRIMARY KEY AUTOINCREMENT, " +   
			" date TEXT, " +   
			" why TEXT, " +   
			" mood NUMERIC" +   
			")";  
			createStmt.text = sql;  
			createStmt.execute(); 
		}
		private static function setupPrefsTable():void {
			// create the db tables if necessary
			try {
			var createStmt:SQLStatement = new SQLStatement();  
			createStmt.sqlConnection = mmDB;  
			var sql:String =   
			"CREATE TABLE IF NOT EXISTS prefs (" +  
			//" id INTEGER PRIMARY KEY AUTOINCREMENT, " +   
			" paramname TEXT PRIMARY KEY, " +   
			" val TEXT " +
			")";  
			createStmt.text = sql;  
			createStmt.execute(); 
			} catch (error:Error) {
				trace("error:", error.toString());
			}
		}
		
		public static function getPref(name:String):String {
			// get a preference from the DB table
			selectPrefStmt.parameters[":paramname"] = name;
			selectPrefStmt.execute();
			var res:SQLResult = selectPrefStmt.getResult();
			if (res && res.data)
				return res.data[0].val;
			else
				return null;
		}
		public static function setPref(name:String, val:String):void {
			// set a preference in the DB table
			// Create a new prefs insertion statement every time cos multiple will be sent at once, so since its async we can't easily reuse it
			var addPrefStmt:SQLStatement = new SQLStatement();    
			var sql:String = "REPLACE INTO prefs (paramname, val)" +  
				"VALUES (:paramname, :val)";  
			addPrefStmt.sqlConnection = mmDBA;  
			addPrefStmt.text = sql;
			addPrefStmt.parameters[":paramname"] = name;
			addPrefStmt.parameters[":val"] = val;
			addPrefStmt.addEventListener(SQLEvent.RESULT, function(e:SQLEvent):void {trace("saved param:", name);});
			addPrefStmt.execute();
			trace('executing setPref with:', name);
		}
		
		public static function postMood(mood:Object):void {
			// post mood into a new db row
			for (var col:String in mood) {
				addMoodStmt.parameters[":"+col] = mood[col];		// SQL needs the :param, but AS doesn't let me create an attribute of Object with the :
			}
			var x:Object = addMoodStmt.parameters;
			addMoodStmt.execute(); 
		}
		public static function getMoodData():Array {
			return result.data.concat();
		}
	
		private static function createSyncStmts():void {
			// Precreate SQL statements to retieve prefs
			trace('creating sync statements');
			var sql:String = "SELECT val FROM prefs WHERE paramname = :paramname";  
			selectPrefStmt.sqlConnection = mmDB;  
			selectPrefStmt.text = sql;  
		}
		private static function createAsyncStmts(event:SQLEvent):void  
			// Precreate the async SQL statements for update or moods and prefs, and mood queries
		{  
			trace("Creating ASync statements"); 
			var sql:String =   
				"INSERT INTO moods (date, why, mood)" +  
				"VALUES (:date, :why, :mood)";  
			addMoodStmt.sqlConnection = mmDBA;  
			addMoodStmt.text = sql;  
			addMoodStmt.addEventListener(SQLEvent.RESULT, executeSelectMoods);		// always update the results data list after an insertion
			var sql2:String = "SELECT mood, why, date FROM moods";
			selectMoodStmt.sqlConnection = mmDBA;
			selectMoodStmt.text = sql2;
			selectMoodStmt.addEventListener(SQLEvent.RESULT, updateResults);
			selectMoodStmt.execute();												// prepopulate the results data set so its ready when needed
			
		}  
		private static function executeSelectMoods(e:Event):void{
			// select is called on completion of an insertion event so that the result set is always ready
			selectMoodStmt.execute();
		}
		private static function updateResults(e:Event):void {
			result = selectMoodStmt.getResult();
		}

	}
}