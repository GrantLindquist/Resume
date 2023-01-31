# Overview

Hexbar Fitness is a full-stack web application using React with Express as a back-end and SQL for database implementation. It features an exercise calendar that utilizes React components to display a clean and functional front-end that updates upon user interaction. It is connected to a MySQL database using a REST API.

![hexbarfitness-homepage](https://user-images.githubusercontent.com/78822631/215652745-f0779e24-0491-4be0-b6f2-a5e77819657f.jpg)

---

# Design Documentation

### UI Sitemap

![image](https://user-images.githubusercontent.com/78822631/215654721-6183fa38-489f-43fd-86e8-645f909b5624.png)


### UI Wireframes

![image](https://user-images.githubusercontent.com/78822631/215654707-5a092b00-4549-47aa-9aec-e230a547f1ec.png)


### UML Diagrams

![image](https://user-images.githubusercontent.com/78822631/215654668-e523c60a-f734-4baa-ac56-6e37483f4acd.png)

---

# Code Snippets

### Overhead App React Component
*note: Since this app does not support user authentication, ?userId=3 represents a default user object that is used for debugging the app. If I were to further develop this app, ?userId=3 would be replaced with the user id of the logged-in user.*

```
const App = () => {

    const [user, setUser] = useState([]);
    // Convert date to string whenever interacting with database
    const [date, setDate] = useState(new Date(Date.now()));

    // Fetches current user JSON
    const loadUser = async() =>{
        const response = await userData.get('?userId=3');
        setUser(response.data);
    }

    // Loads users upon component render
    useEffect(() => {
        loadUser();
    // eslint-disable-next-line react-hooks/exhaustive-deps
    }, []);

    // Handles click event on calendar cell - takes string parameter
    const handleClick = (date) => {
        setDate(new Date(date));
    }

    // Applies user to home page
    var homePage = user.map((user) => {
        return(<>       
            <BrowserRouter>
                <Navbar/> 
                <Routes>
                    <Route path='/' element={<>
                        <Aside currentDate={date} firstName={user.firstName} handleClick={handleClick}/>
                        <Main key={date} currentDate={date}/>
                    </>}></Route>
                    <Route path='/profile' element={<Profile user={user}/>}></Route>
                </Routes>
            </BrowserRouter>
        </>)
    });
    return <>{homePage}</>
};

export default App;
```

### React Form that creates a new Exercise object

```
const ExerciseForm = (param) => {
  
  // Blank exercise object to place in form
  var exercise = {
    userId: 3,
    name: '',
    date: moment(param.currentDate).format('YYYY-MM-DD'),
    sets: '',
    reps: '',
  };
  if(!param.isCreatingNewExercise){
    exercise = {
      userId: 3,
      exerciseId: param.exerciseId,
      name: param.exercise.name,
      date: exercise.date,
      sets: param.exercise.sets,
      reps: param.exercise.reps,
    };
  }

  // States for handling exercise creation/revision
  const [exerciseName, setExerciseName] = useState('');
  const [exerciseSets, setExerciseSets] = useState(0);
  const [exerciseReps, setExerciseReps] = useState(0);

  const updateExerciseName = (event) => {
    setExerciseName(event.target.value);
  };
  const updateExerciseSets = (event) => {
    setExerciseSets(event.target.value);
  };
  const updateExerciseReps = (event) => {
    setExerciseReps(event.target.value);
  };

  // Submits exercise to API
  const handleSubmit = (event) => {
    event.preventDefault();
    var newExercise = {
      userId: exercise.userId,
      exerciseId: exercise.exerciseId,
      name: exerciseName,
      date: exercise.date,
      sets: exerciseSets,
      reps: exerciseReps
    };
    submitExercise(newExercise);
  };

  // Places exercise into API
  const submitExercise = async (exercise) => {
    let response;
    if (param.isCreatingNewExercise) {
      response = await exerciseData.post("?userId=3", exercise);
    } else {
      response = await exerciseData.put(`?userId=3&exerciseId=${exercise.exerciseId}`, exercise);
    }
    console.log(response);
    param.refresh();
    param.closeModal();
  };

  return (
    <div id="hexbar-exercise-form" className="hidden">
      <div className="container p-3 text-center">
        <button id='close-modal-btn' className='btn-close btn-close-white' onClick={() => param.closeModal()}></button>
        <h3 className='pt-4'>{param.isCreatingNewExercise ? "Create" : "Update"} Exercise</h3>
        <form className="py-2 px-5 mx-5" onSubmit={handleSubmit}>
          <div className="form-floating">
            <label>Name</label>
            <input className="form-control" maxLength='30' placeholder="Exercise Name" onChange={updateExerciseName} ></input>
          </div>
          <div className='d-flex justify-content-center'>
            <div className="form-floating">
              <input placeholder="Sets" name="sets" className="form-control" onChange={updateExerciseSets}></input>
              <label>Sets</label>
            </div>
            <div className="form-floating">
              <input placeholder="Reps" name="reps" className="form-control" onChange={updateExerciseReps}></input>
              <label>Reps</label>
            </div>
          </div>
          <button className="btn btn-secondary mt-4" type="submit">Submit</button>
        </form>
      </div>
    </div>
  );
};
export default ExerciseForm;
```

### User object Express controller
*note: Since this app does not support user authentication, ?userId=3 represents a default user object that is used for debugging the app. If I were to further develop this app, ?userId=3 would be replaced with the user id of the logged-in user.*

```
// Inserts exercises into user
async function getExercises(users: User[], res: Response<any, Record<string, any>>) {
  for (let i = 0; i < users.length; i ++) {
    try {
      let exercises = await ExerciseDAO.getExercises(users[i].userId);
      users[i].exercises = exercises;

    } catch (error) {
      console.error ('[user.controller] [getExercises] [Error]', error );
      res.status(500).json({
       message: 'There was an error when fetching exercises'
      });
    }
  }
};

// Creates user and places into DB
export const getExercisesByDate: RequestHandler = async (
  req: Request,
  res: Response
) => {
  try {
    let userId = parseInt(req.query.userId as string);
    let date = req.query.date as string;

    // Create user in DB
    const exercises = await ExerciseDAO.getExercisesByDate(userId, date);
    
    // Returns OK
    res.status(200).json(exercises);
  } 
  // If error
  catch (error) {
    // Log error 
    console.error("[user.controller] [getExercisesByDate] [Error]", error);
    // Send 500 error
    res.status(500).json({message: " There was an error when getting an exercises"});
  }
};

// Creates user and places into DB
export const createExercise: RequestHandler = async (
  req: Request,
  res: Response
) => {
  try {
    // Create user in DB
    const exercise = await ExerciseDAO.createExercise(req.body);  

    // Returns OK
    res.status(200).json(exercise);
  } 
  // If error
  catch (error) {
    // Log error 
    console.error("[user.controller] [createExercise] [Error]", error);
    // Send 500 error
    res.status(500).json({message: " There was an error when creating an exercise"});
  }
};

// Updates exercise info in DB
export const updateExercise: RequestHandler = async (
  req: Request,
  res: Response
) => {
  try {
    // Executes update query
    let exerciseId = parseInt (req.query.exerciseId as string ) ;
    const exercise = await ExerciseDAO.updateExercise(req.body, exerciseId);
    res.status(200).json(exercise);
  } 
  catch (error) 
  {
    console.error("[user.controller] [updateExercise] [Error]", error);
    res.status(500).json({
      message: "An error occurred while updating exercise",
    });
  }
};

// Deletes exercise from DB
export const deleteExercise: RequestHandler = async (
  req: Request,
  res: Response
) => {
  try {
    // Executes update query
    let exerciseId = parseInt (req.query.exerciseId as string ) ;
    const response = await ExerciseDAO.deleteExercise(exerciseId);
    res.status(200).json(response);
  } 
  catch (error) 
  {
    console.error("[user.controller] [deleteExercise] [Error]", error);
    res.status(500).json({
      message: "An error occurred while deleting exercise",
    });
  }
};

// Gets users from DB
export const getUsers: RequestHandler = async (
    req: Request,
    res: Response
  ) => {
    try {
      let user;
      let userId = parseInt(req.query.userId as string);

      // If there is no user id requested
      if (Number.isNaN(userId)) {
        // Return every user from DB
        user = await UserDAO.getUsers();
      } 
      // If user id is specified
      else {
        // Return user of specified id
        user = await UserDAO.getUserByUserId(userId);
      }
      // Fill user with exercise objects
      await getExercises(user, res);
  
      // Return OK status with user JSON
      res.status(200).json(user);
    } 
    // If any error occurs
    catch (error) {
      // Log error
      console.error("[user.controller] [getUsers] [Error]", error);
      // Send 500 server error
      res.status(500).json({
        message: "Error occurred while fetching users",
      });
    }
  };

// Gets user by username using LIKE query
export const getUsersByUsernameSearch: RequestHandler = async (
    req: Request,
    res: Response
  ) => {
    try {
      // Executes search query using username parameter
      const user = await UserDAO.getUsersByUsernameSearch(
        "%" + req.params.search + "%"
      );

      // Fill user with exercise objects
      await getExercises(user, res);

      // Return OK status with user JSON
      res.status(200).json(user);
    }
    // If error occurs 
    catch (error) {
      // Log error
      console.error("[user.controller] [getUsers] [Error]", error);
      // Send 500 server error
      res.status(500).json({
        message: "Error occurred while fetching users",
      });
    }
  };

// Creates user and places into DB
export const createUser: RequestHandler = async (
    req: Request,
    res: Response
  ) => {
    try {
      // Create user in DB
      const okPacket: OkPacket = await UserDAO.createUser(req.body);  
      // Creates exercise objects for user
      req.body.exercises.forEach(async (exercise: Exercise) => {
        try {
          // Executes create query
          await ExerciseDAO.createExercise(exercise);
        } 
        // If error
        catch (error) {
          // Log error
          console.error("[user.controller] [createExercises] [Error]", error);
          // Send 500 error
          res.status(500).json({message: "An error occurred while creating exercises"});
        }
      });
      // Returns OK
      res.status(200).json(okPacket);
    } 
    // If error
    catch (error) {
      // Log error 
      console.error("[user.controller] [createUser] [Error]", error);
      // Send 500 error
      res.status(500).json({message: " There was an error when creating users"});
    }
  };
  
// Updates user info in DB
export const updateUser: RequestHandler = async (
    req: Request,
    res: Response
  ) => {
    try {
      // Executes update query
      const okPacket: OkPacket = await UserDAO.updateUser(req.body);
      res.status(200).json(okPacket);
    } 
    catch (error) 
    {
      console.error("[user.controller] [updateUser] [Error]", error);
      res.status(500).json({
        message: "An error occurred while updating user",
      });
    }
  };
  
// Deletes user from DB
export const deleteUser: RequestHandler = async (req: Request , res: Response) => {
    try {
      let userId = parseInt (req.params.userId as string ) ;
      // Deletes exercises in user
      await ExerciseDAO.deleteExercisesByUserId(userId);
      const response = await UserDAO.deleteUser(userId) ;
      res.status(200).json(response);
    } 
    catch ( error ) {
      console.error('[albums.controller] [deleteUser] [Error]', error);
      res.status(500).json({
       message : 'There was an error when deleting the user'
      });
    }
  };
```

### DDL Scripts

```
DROP DATABASE IF EXISTS hexbar;
CREATE DATABASE  IF NOT EXISTS `hexbar` /*!40100 DEFAULT CHARACTER SET utf8 */;
USE `hexbar`;
-- MySQL dump 10.13  Distrib 5.7.26, for osx10.10 (x86_64)
--
-- Host: localhost    Database: hexbar
-- ------------------------------------------------------
-- Server version	5.7.26

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
--
-- Table structure for table `hexbar`
--

DROP TABLE IF EXISTS `user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `user` (
  `userId` INT(12) NOT NULL AUTO_INCREMENT,
  `firstName` VARCHAR(30),
  `lastName` VARCHAR(30),
  `username` VARCHAR(30) NOT NULL,
  `password` VARCHAR(30) NOT NULL,
  PRIMARY KEY (`userId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES 
	(3, 'Grant', 'Lindquist', 'grantlindquist', 'root');
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;

--
-- Table structure for table `exercise`
--

DROP TABLE IF EXISTS `exercise`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `exercise` (
  `exerciseId` INT(12) NOT NULL AUTO_INCREMENT,
  `userId` INT(12) NOT NULL,
  `name` VARCHAR(30) NOT NULL,
  `date` DATE NOT NULL,
  `sets` INT(10) NOT NULL,
  `reps` INT(50) NOT NULL,
  PRIMARY KEY (`exerciseId`),
  KEY `user_id_FK_idx` (`userId`),
  CONSTRAINT `user_id_FK` FOREIGN KEY (`userId`) REFERENCES `user` (`userId`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `tracks`
--

/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed
```
