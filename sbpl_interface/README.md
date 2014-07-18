# SBPL for MoveIt

This creates a (hopefully) cleaner interface to SBPL for 7-DOF arm planning.

## Required Components
 * PlanningData
   * This holds the actual graph, in the form of a hash table, can be taken as is.
 * BFS
 * Environment (plus statistics and parameters)
   * Needs access to planning_scene, robot_state
   * Create distance field from robot state for evaluating cell cost - C_cell(s')
   * Run BFS for H_xyz(s). 2010 paper talks about 100^3 grid for BFS taking 0.4s to compute.
 * Motion Primitives
   * Simple n-tuples for n joints
     * sbpl_motion_planning seems to use same prims for both short and long distance
     * long distance only uses the upper 4 joints of the arm
     * short distance might be good with 4 joints in more open spaces, needs all joints in more cluttered space.
   * Snap to XYZRPY (do ik when within predefined distance of goal)
     * The distance to goal is computed from the BFS distance so that it includes obstacles.
     * Distance threshold is specified as 10cm in ICRA2011 paper.
   * Snap to RPY (orientation solver)
     * This doesn't appear to be used at first look in sbpl_motion_planning, however,
       it's baked into the computeIK call.
   * Loader utilities to create from file
 * Add support for joint-space goals?

## Other Notes

Cost:

    C(s,s') = C_cell(s') + W_action * C_action(s,s') + W_smooth * C_smooth(s,s')

Heuristic:

    H(s) = H_xyz(s) + w*H_rpy(s)

ROS MoveIt Existing Interface
 * !use_standard_collision_checking = causes planner to use the CollisionWorldHybrid (which is now in experimental).
   See (https://github.com/ros-planning/moveit_core/commit/e1cb349ecfd2dad8b61d1b0d3717036175ce61ba).
   Also disallows the use of BFS (which makes the planner really slow).
   Although it is duplicating data and not nearly as nice -- maybe we just use the regular collision world and get rid of this param?
 * 