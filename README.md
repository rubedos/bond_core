# bond_core extended functionality
Forked from ROS bond_core for additional functionality: detect stalled main loop of the node (manual pulse).
The original ROS Bond can detect if two bound nodes are alive, i.e. if either of them crashes the sister node is notified about it and can take preventive actions, like stop the robot.
However there is another critical situation which is not detected - that one of the mainloops of the sister nodes has stalled, either because of a dead-lock or too long calculations. It happens because Bond has internal timer running on it's own thread which keeps notifying sister node despite main thread being blocked from execution. To resolve this matter it is necessary to extend Bond API by requiring to manually call pulse() method from nodes main loop, which is the only guarantee that nodes main-loop is still processing at desired frequency.

To keep original functionality, Bond API works just like it was designed and to get the new functionality Bond clients must do some little effort - call Bond::setManualPulse(true) to mark that heartbeat pulses shall be invoked from mainloop rather than internal timer. And of course - add bond::pulse() to the main-loop:
```
{
	// Critical scope
	Bond bond("my_id");
	bond.setManualPulses(true);	// mark that host node will do it's own pulsing
	rate r(hz);
	while (ros::ok())
	{
		// Do heavy calculations
		bond.pulse();	// indirectly calls doPublishing(). Failing to call it means node has stalled or crashed
	}
}
```
