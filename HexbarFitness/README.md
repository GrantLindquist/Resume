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
*note: Since this app does not support user authentication, ?userId=3 represents a default user object that is used for debugging the app. If I were to further develop this app, ?userId=3 would be replaced with the user id of the logged-in user.*

### Overhead App React Component
The root React Component for this project. Demonstrates React Hooks and utilizing React States that reload the component if necessary.

```
const App = () => {

    // User State - represents user that is signed into application.
    const [user, setUser] = useState([]);
    // Date State - reresents the date that user is viewing exercises from. By default, it is set to the current date.
    const [date, setDate] = useState(new Date(Date.now()));

    // Fetches JSON data from signed-in user.
    const loadUser = async() =>{
        const response = await userData.get('?userId=3');
        setUser(response.data);
    }

    // Loads users upon component render.
    useEffect(() => {
        loadUser();
    // eslint-disable-next-line react-hooks/exhaustive-deps
    }, []);

    // Handles click event on calendar cell - accepts string parameter that represents new date.
    const handleClick = (date) => {
        setDate(new Date(date));
    }

    // Maps User State data to home page.
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
This React form takes user input, creates an Exercise object using the input, and submits an API request to place the Exercise inside of the database. The component reloads upon submitting to seamlessly add the Exercise into the user interface.

```
const ExerciseForm = (param) => {
  
  // Blank exercise object that form will fill-out.
  var exercise = {
    userId: 3,
    name: '',
    date: moment(param.currentDate).format('YYYY-MM-DD'),
    sets: '',
    reps: '',
  };
  // If user is updating an Exercise rather than creating one, the information from the Exercise being updated is automatically used.
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

  // State-changing methods
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
    // Calls POST request if user is creating a new Exercise
    if (param.isCreatingNewExercise) {
      response = await exerciseData.post("?userId=3", exercise);
    } 
    // Calls PUT request if user is updating a previous exercise
    else {
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
