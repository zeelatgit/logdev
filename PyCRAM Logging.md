


# Task Tree in PyCRAM
PyCRAM uses AnyTree to perform various tasks using task trees. This task tree method is defined in the file 'task.py'. The file defines the task tree structure and the task tree classes:
1.  **TaskTreeNode Class**: This class represents a node in the task tree. Each node contains information about the task to be executed, its status, start and end times, and optional reason for failure.
    
2.  **Code Class**: This class represents an executable code block in a plan. It holds a reference to a function or method and its associated keyword arguments.
    
3.  **NoOperation Class**: This class represents a convenience class for a no-operation task. It is used when no specific task is assigned to a node.
    
4.  **with_tree Decorator**: This decorator records the function name, arguments, and execution metadata in the task tree. It is used to wrap functions/methods that represent tasks, allowing them to be automatically added to the task tree when executed.
    
5.  **SimulatedTaskTree Class**: This class represents a task tree for execution in a simulated environment. It provides a context manager interface to suspend the current task tree and create a new simulated task tree for execution within a simulation.
    
6.  **reset_tree Function**: This function resets the current task tree to an empty root node.

For the logging interface, we can store information from the task tree such as the name of the function being executed, and the arguments used with it. More information that we can log from here, includes the status of the task tree node, its star time and its end time and its node id and parent node id.
For the logging, we decided to store all this data into a JSON dictionary. Some of the classes in the task tree have method defined under them, **to_json** method, for the data to be serialized into a JSON format. But some classes did not have an existing **to_json** method linked to them. We would also need to define these methods for each designator class, so that all the actions that are being executed are serialized into a JSON format.

We can find an already existing **to_json** method defined in the Code Class (serializes the function name and keyword arguments into a JSON form) for both the function and the keyword arguments. We also have such a method defined under the **TaskTreeeNode Class**, which serializes the status, the node id and the parent node id. We will add one more field here, **reason_of_failure** which will state the reason of failure of an action, if there exist one.
So the new **to_json** method under **class TaskTreeNode** will look like this:

    def to_json(self):  
        return {"code": self.code.to_json(),  
                "status": self.status.name,  
                "start_time": self.start_time.isoformat() if self.start_time else None,  
                "end_time": self.end_time.isoformat() if self.end_time else None,  
                "id": id(self),  
                "parent_id": id(self.parent) if self.parent else None,  
                "reason_of_failure": str(self.reason) if self.reason else None  
      }

# Action and Motion Designators
Now we will start defining the JSON serialization methods for each action designators, as action designators represent various actions that can be performed by a robot agent.

We write down the **to_json** methods for all the different action designator classes in the following way:

**class MoveTorsoAction:**
  
        def to_json(self) -> dict:  
            """Converts the instance to a JSON-compatible dictionary."""  
             return {  
                "action_type": "MoveTorsoAction",  
                "position": self.position,  
            }

 **class SetGripperAction:**

    def to_json(self) -> dict: 
    """  Convert the action to a JSON-compatible dictionary.  """ 
    	    return {  
    	        "action_type": "SetGripperAction",  
    	        "gripper": self.gripper,  
    	        "motion": self.motion  
    	    }
**class ReleaseAction:**

    def to_json(self) -> dict:  
    """  Convert the action to a JSON-compatible dictionary. """  
	    return {  
	        "action_type": "ReleaseAction",  
	        "gripper": self.gripper,  
	        "object_designator": self.object_designator.to_json()  
	    }

**class GripAction:**

    def to_json(self) -> dict:  
    """  Convert the action to a JSON-compatible dictionary. """  
	    return {  
	        "action_type": "GripAction",  
	        "gripper": self.gripper,  
	        "object_designator": self.object_designator.to_json(),  
	        "effort": self.effort  
	    }

**class ParkArmsAction:**

    def to_json(self) -> dict:  
    """ Convert the action to a JSON-compatible dictionary. """  
	    return {  
	        "action_type": "ParkArmsAction",  
	        "arm": self.arm.name  
	    }
**class PickUpAction:**
In PickUpAction class we add the object_designator in the JSON dictionary only if it is present or initialised.

    def to_json(self) -> dict:  
    """Converts the instance to a JSON-compatible dictionary."""  
	    json_data = {  
	        "action_type": "PickUpAction",  
	        "arms": self.arm,  
	        "grasps": self.grasp,  
	    }  
	    if self.object_designator:  
	        json_data["object_designator_description"] = self.object_designator.to_json()  
	    return json_data

**class PlaceAction:**
In this class we define two JSON methods for the PlaceAction class itslef and the nested **Action** class inside this.

    def to_json(self) -> dict:  
        """Converts the instance to a JSON-compatible dictionary."""  
	      return {  
	            "action_type": "PlaceAction",  
	            "object_designator": self.object_designator.to_json(),  
	            "arm": self.arm,  
	            "target_location": self.target_location.to_json(),  
            
      }

**class NavigateAction:**

    def to_json(self) -> dict:  
        """Converts the instance to a JSON-compatible dictionary."""  
	     return {  
            "action_type": "NavigateAction",  
            "target_location": self.target_location.to_json(),  
      }

**class TransportAction:**

    def to_json(self) -> dict:  
     """  
     Convert the action to a JSON-compatible dictionary. 
     """  
	    return {  
            "action_type": "TransportAction",  
            "object_designator": self.object_designator.to_json(),  
            "arm": self.arm,  
            "target_location": self.target_location.to_json()  
        }

**class LookAtAction:**

    def to_json(self) -> dict:  
     """  
     Convert the action to a JSON-compatible dictionary. 
     """  
	    return {  
            "action_type": "LookAtAction",  
            "target": self.target.to_json()  
        }
**class DetectAction:**

    def to_json(self) -> dict:  
     """  
     Convert the action to a JSON-compatible dictionary.
     """  
	    return {  
            "action_type": "DetectAction",  
            "object_designator": self.object_designator.to_json()  
        }
        
**class OpenAction:**

    def to_json(self) -> dict:  
        """  
     Convert the action to a JSON-compatible dictionary. 
     """  
	     return {  
            "action_type": "OpenAction",  
            "object_designator": self.object_designator.to_json(),  
            "arm": self.arm  
        }
        
**class CloseAction:**

    def to_json(self) -> dict:  
     """  
     Convert the action to a JSON-compatible dictionary. 
     """  
	     return {  
            "action_type": "CloseAction",  
            "object_designator": self.object_designator.to_json(),  
            "arm": self.arm
         }

**class GraspingAction:**

    def to_json(self) -> dict:  
     """  
     Convert the action to a JSON-compatible dictionary.
     """  
	    return {  
            "action_type": "GraspingAction",  
            "arm": self.arm,  
            "object_desig": self.object_desig.to_json()  
        }
##
Similarly, we can write **to_json** methods for the motion designators. Motion designators are required to be JSON serialized because motion designators are similar to action designators except that they are lower level representation of actions.
This will enable us to log the data about the motions involved in executing an action.

# Designator and Pose Files
We will also need to write a **to_json** method inside the 'designator.py' file, under the class **ObjectDesignatorDescription**.  This class defines another class inside it, **class Object**. The object designator description will basically be initialised from here. Since, the Object designator interacts with the action designators, we will need to write a JSON serialization method for this class.

    def to_json(self) -> dict:  
     """  
     Convert the object to a JSON-compatible dictionary. 
     """  
	    return {  
            "name": self.name,  
            "type": self.type,  
            "pose": self.pose.to_json() if hasattr(self, 'pose') else None
        }

Now we will, add a JSON serialization method for the **class Pose**. This is needed because the poses and orientation of different objects are initialised under this class.

    def to_json(self):  
        return {  
            "position": self.position_as_list(),  
            "orientation": self.orientation_as_list(),  
            "frame": self.frame  
        }

# Modified @with_tree Decorator

Finally we will modify the code for the @with_tree decorator, to use the JSON serialization methods we wrote. And then, we will send this data to an API where it can be stored. For this, we will also define another function **send_json**, before the modified @with_tree decorator, to send the JSON dictionaries we create.

   

    def send_json(json_info):  
	    """Send JSON info to the API."""  
	    API_URL = "http://localhost:5000/neemlog/api/log"  
	    headers = {'Content-Type': 'application/json'}  # Headers specifying JSON content  
      
	    # Send HTTP POST request with JSON data  
	    response = requests.post(API_URL, json=json_info, headers=headers)  
      
	    # Check if the request was successful (status code 200)  
		    if response.status_code == 200:  
	    print("JSON info sent successfully to the API.")  
	    else:  
		    print(f"Failed to send JSON info to the API. Status code: {response.status_code}")  
		    print(response.text)  # Print the error message if available

-----------------------------------------------------------------------------------------------

    def with_tree(fun: Callable) -> Callable:  
    """
    Decorator that records the function name, arguments and execution metadata in the task tree.  
      
    :param fun: The function to record the data from. 
    """  
	    def handle_tree(*args, **kwargs):  
      
		    # get the task tree  
		    global task_tree  
      
		    # create the code object that gets executed  
		    code = Code(fun, inspect.getcallargs(fun, *args, **kwargs))  
      
		    task_tree = TaskTreeNode(code, parent=task_tree)  
      
		    # Print information about the executable code  
		    json_info = task_tree.to_json()
		    send_json(json_info)
		    #
		    # Try to execute the task  
  

		    try:  
			    task_tree.status = TaskStatus.CREATED  
			    task_tree.start_time = datetime.datetime.now()  
			    result = task_tree.code.execute()  
		      
			    # if it succeeded set the flag  
			    task_tree.status = TaskStatus.SUCCEEDED  
		      
			    json_info = task_tree.to_json()  
			    print(f"JSON Info: {json_info}")  
		      
		    # iff a PlanFailure occurs  
		    except PlanFailure as e:  
		      
			    # log the error and set the flag  
			    logging.exception("Task execution failed at %s. Reason %s" % (str(task_tree.code), e))  
			    task_tree.reason = e  
			    task_tree.status = TaskStatus.FAILED  
			    raise e  
		      
				json_info["reason_of_failure"] = str(e)  
		      
		    finally:  
			    # set and time and update current node pointer  
			    task_tree.end_time = datetime.datetime.now()  
			    task_tree = task_tree.parent  
		    return result  
		      
		json_info = task_tree.to_json()  
		print(f"JSON Info: {json_info}")
		    
		return handle_tree

**Note:** I added a more than one instances of **json_info** dictionary within the @with_tree decorator definition, because upon placing them at different locations within the @with_tree decorator, would affect the various information being logged in our JSON dictionary (some of the fields, such as the start_time and task tree status, would be empty). All the three instances added generate different outputs. For example, the last instance, right before 'return handle_tree' line, would return an output for 'No Operation' as well. The instance present inside the Try block would generate output for task tree nodes only when the status is 'SUCCEEDED'. The first instance of **json_info** would return outputs also for the times right when it is 'CREATED'. Therefore, I left all the three instances at their place.

