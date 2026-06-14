
# 🐕 Layer 1: What's Already Built Into the Dog (The "Easy Start")

For simple following tasks, the Go2 has a built-in feature called ISS (Intelligent Side Follow) . With just a few button presses, the robot will lock onto a person and follow them .

How it works: You stand next to the robot, and it uses its sensors to maintain a set distance while mirroring your speed .
The huge limitation: This built-in feature is designed for a "pack leader"—an adult it can see and track. It is not a general "escort mode" and would be unreliable for following a child, who might be too short, move erratically, or get blocked by obstacles. It also has no memory or map, so it can't know the way to school on its own.
🤖 Layer 2: The Custom "Follow My Kid" Program (What You Will Build)

To solve the first problem (reliably following your child), you would write a ROS 2 program that replaces Unitree's built-in follow mode. This is a publisher/subscriber system, just like the talker-listener demo.

You wouldn't be controlling the 20 leg joints directly. Instead, you'd write a vision node (the "Brain") that publishes high-level commands like Move(0.5 m/s, turn left) to the robot's built-in motion control node (the "Legs"), which handles the complex stability.

The table below shows exactly how the talker-listener pattern translates to your custom follower :

Your New Node (The "Brain")	ROS Topic (The "Nerve")	The Built-in Node (The "Legs")
What it does: Processes the camera feed to find and track your kid.	Name: /cmd_vel (Command Velocity)
Message Type: Twist (contains linear & angular speeds)	What it does: Receives speed commands and translates them into precise joint movements for walking.
It PUBLISHES a velocity command to this topic.	This is the communication channel.	It SUBSCRIBES to this topic to get its marching orders.
The core of your program would be:

python
# Subscribe to camera feed
camera_sub = self.create_subscription(Image, '/camera/feed', self.camera_callback)

# Inside the callback: Detect kid and calculate commands

if kid_is_too_far_right:
    cmd = Twist()                # Create a velocity message
    cmd.angular.z = -0.5         # Turn right
    self.velocity_publisher.publish(cmd) # Send command to legs!
The search results show that a similar system was built for the Go2, using YOLOv8 (an AI object detection model) to track a person and send velocity commands to the robot .

🗺️ Layer 3: The School Route Navigation (The "Full Autonomy")

This is the most complex part. The "following" mode requires the child to be present. For true escorting, the robot needs to know the route to school even if the child is not there to lead it. This adds two major components to the system you would build:

SLAM (Simultaneous Localization and Mapping): You would walk the route with the robot once, and it would use its LiDAR and camera to create a 3D map of the sidewalks, crosswalks, and turns . This is its "mental map."
Nav2 (ROS 2 Navigation Stack): This is a powerful system that uses the map to plan a safe path. It handles dynamic obstacle avoidance (e.g., walking around a parked car or another pedestrian) while staying on the pre-planned route .
Academic teams have demonstrated similar concepts using the Go2 for autonomous navigation on complex courses, though they required additional sensors for robustness .

💎 Summary: What You Will and Won't Write

What you DO write: The high-level "brain" software. For the "escort" feature, you'd write a vision-based follower (Layer 2). For full autonomy, you'd integrate the SLAM and Nav2 systems (Layer 3).
What you DON'T write: The low-level leg control, joint stabilization, motor drivers, and basic walking gaits. Unitree has done that incredible engineering work for you, which is what makes the Go2 such a powerful platform.
Building a system that can safely escort a child on a public sidewalk involves solving some of the hardest problems in robotics (navigation, dynamic obstacle avoidance, fail-safe reliability). The factory "follow mode" is a fun toy, but using the Go2's SDK to write your own ROS 2 follower is the exact path to turning it into a real tool.

Starting with the "custom follower" (Layer 2) using the go2_ros2_sdk  would be a perfect first major project. Would you like to see a more detailed code outline for that vision-based follower node?
