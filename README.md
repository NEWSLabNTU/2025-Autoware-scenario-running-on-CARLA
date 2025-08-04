# The integration of Tier IV evaluator and CARLA

## Requirements

1. Assume that you have installed CARLA 0.9.15 and Tier IV Evaluator in your home directory.

## Convert .yaml scenario to .xosc format

1. Run the following command to test a scenario:
```
ros2 launch scenario_test_runner scenario_test_runner.launch.py \
architecture_type:=awf/universe/20250130 \
record:=false \
scenario:='/path/to/your/scenario.yaml' \
sensor_model:=sample_sensor_kit \
vehicle_model:=sample_vehicle
```
2. The converted scenario will be in `/tmp/scenario_test_runner/YOUR_SCENARIO_NAME`. There may be one or more files.

## Comaptibility between Tier IV evaluator and CARLA
1. Change the map to OpenDrive format because CARLA only accept OpenDrive format now.
2. Modify evaluator criteria:
```xml
<UserDefinedAction>
  <CustomCommandAction type="exitSuccess" />
</UserDefinedAction>
```
to
```xml
<UserDefinedAction>
  <CustomCommandAction type="python exitSuccess.py" />
</UserDefinedAction>
```
because CARLA expect a command in `CustomCommandAction`.

3. Make sure the format is compatible to OpenSCENARIO 1.0. Below is a reference of the compatibility between scenario simulator v2 and CARLA.
https://docs.google.com/spreadsheets/d/18nD7_fi7StzBdqiFsIotrp21MWozj5fLqcInsV_-RZY/edit?gid=1967977418#gid=1967977418

5. Below is an example of modified scenario:
```xml
<?xml version="1.0"?>
<OpenSCENARIO>
	<FileHeader author="Tatsuya Yamasaki" date="2022-03-04T18:06:53+09:00" description="Sample scenario (with Autoware)" revMajor="1" revMinor="0" />
	<ParameterDeclarations />
	<CatalogLocations>
		<!-- Use the car supported by CARLA-->
		<VehicleCatalog>
			<Directory path="catalogs" />
		</VehicleCatalog>
	</CatalogLocations>
	<RoadNetwork>
		<!-- Map limitation -->
		<!-- <LogicFile filepath="$(ros2 pkg prefix \-\-share kashiwanoha_map)/map" /> -->
		<LogicFile filepath="Town01" />
	</RoadNetwork>
	<Entities>
		<ScenarioObject name="ego">
			<!-- Use the car supported by CARLA-->
			<CatalogReference catalogName="VehicleCatalog" entryName="vehicle.volkswagen.t2"/> 
			<ObjectController>
				<Controller name="Autoware">
					<Properties>
						<Property name="maxJerk" value="1.5" />
						<Property name="minJerk" value="-1.5" />
					</Properties>
				</Controller>
			</ObjectController>
		</ScenarioObject>
	</Entities>
	<Storyboard>
		<Init>
			<Actions>
				<Private entityRef="ego">
					<PrivateAction>
						<TeleportAction>
							<Position>
								<!-- Map modification -->
								<LanePosition roadId="4" laneId="-1" s="48.58">
									<Orientation type="relative" h="0" p="0" r="0" />
								</LanePosition>
							</Position>	
						</TeleportAction>
					</PrivateAction>
					<PrivateAction>
						<RoutingAction>
							<AcquirePositionAction>
								<Position>
								 <!-- Map modification -->
									<LanePosition roadId="4" laneId="-1" s="30">
										<Orientation type="relative" h="0" p="0" r="0" />
									</LanePosition>
								</Position>
							</AcquirePositionAction>
						</RoutingAction>
					</PrivateAction>
				</Private>
			</Actions>
		</Init>
		<Story name="">
			<Act name="_EndCondition">
				<ManeuverGroup maximumExecutionCount="1" name="">
					<Actors selectTriggeringEntities="false">
						<EntityRef entityRef="ego" />
					</Actors>
					<Maneuver name="">
						<Event name="success exit" priority="parallel">
							<Action name="success exit action">
								<UserDefinedAction>
									<!-- Use command to run UserDefinedAction -->
									<CustomCommandAction type="python exitSuccess.py" />
								</UserDefinedAction>
							</Action>
							<StartTrigger>
								<ConditionGroup>
									<Condition name="123" delay="0" conditionEdge="none">
										<ByEntityCondition>
											<TriggeringEntities triggeringEntitiesRule="any">
												<EntityRef entityRef="ego" />
											</TriggeringEntities>
											<EntityCondition>
												<ReachPositionCondition tolerance="0.5">
													<Position>
													    <!-- Map modification -->
														<!-- scenario editor's original value: <LanePosition roadId="" laneId="34507" s="50" offset="0"> -->
														<LanePosition roadId="4" laneId="-1" s="50">
															<Orientation type="relative" h="0" p="0" r="0" />
														</LanePosition>
													</Position>
												</ReachPositionCondition>
											</EntityCondition>
										</ByEntityCondition>
									</Condition>
								</ConditionGroup>
								<!-- Add another condition group for testing -->
								<ConditionGroup>
									<Condition name="456" delay="0" conditionEdge="none">
										<ByEntityCondition>
											<TriggeringEntities triggeringEntitiesRule="any">
												<EntityRef entityRef="ego" />
											</TriggeringEntities>
											<EntityCondition>
												<ReachPositionCondition tolerance="0.5">
													<Position>
													    <!-- Map modification -->
														<!-- scenario editor's original value: <LanePosition roadId="" laneId="34507" s="50" offset="0"> -->
														<LanePosition roadId="4" laneId="-1" s="40">
															<Orientation type="relative" h="0" p="0" r="0" />
														</LanePosition>
													</Position>
												</ReachPositionCondition>
											</EntityCondition>
										</ByEntityCondition>
									</Condition>
								</ConditionGroup>
							</StartTrigger>
						</Event>
						<Event name="fail exit" priority="parallel">
							<Action name="fail exit action">
								<UserDefinedAction>
									<!-- Use command to run UserDefinedAction -->
									<CustomCommandAction type="python exitFailure.py" />
								</UserDefinedAction>
							</Action>
							<StartTrigger>
								<ConditionGroup>
									<Condition name="" delay="0" conditionEdge="none">
										<ByValueCondition>
											<SimulationTimeCondition value="180" rule="greaterThan" />
										</ByValueCondition>
									</Condition>
								</ConditionGroup>
							</StartTrigger>
						</Event>
					</Maneuver>
				</ManeuverGroup>
				<StartTrigger>
					<ConditionGroup>
						<Condition name="" delay="0" conditionEdge="none">
							<ByValueCondition>
								<SimulationTimeCondition value="0" rule="greaterThan" />
							</ByValueCondition>
						</Condition>
					</ConditionGroup>
				</StartTrigger>
			</Act>
		</Story>
		<StopTrigger />
	</Storyboard>
</OpenSCENARIO>
```
## Run .xosc scenario in CARLA

1. Prepare exitSuccess.py and exitFailure.py. Below is an example of exitSuccess.py:
```python=
import os
import signal

def get_ppid(pid):
    stat = open(f"/proc/{pid}/stat").read().split()
    return int(stat[3])

def kill_grandparent(sig=signal.SIGTERM):
    parent_pid = os.getppid()
    try:
        grandparent_pid = get_ppid(parent_pid)
    except Exception:
        print(f"Read /proc/{parent_pid}/stat fail.")
        return

    try:
        with open(f"/proc/{grandparent_pid}/comm") as f:
            gp_name = f.read().strip()
    except Exception:
        gp_name = "<unknown>"

    print(f"Killing grandparent {grandparent_pid} ({gp_name}) with {sig.name}")
    os.kill(grandparent_pid, sig)

if __name__ == "__main__":
    kill_grandparent()
    print("Exit Success.")
```

2. Use following command to launch CARLA server:
`./CarlaUE4.sh`
3. Use following command to run .xosc scenario:
`python scenario_runner.py --timeout 20 --openscenario /path/to/your/scenario.xosc --reloadWorld [--output] [--debug]`
The option `--output` can print CARLA criteria when a scenario is finished.
The option `--debug` can monitor whether each condition is triggered or not.
4. Use following command to open manual control window:
`python manual_control.py --rolename YOUR_EGO_NAME`
`YOUR_EGO_NAME` is the ego name in your scenario. If `--rolename` is not provided, it takes "hero" as ego name by default.

## Reference

1. Tier IV simulator v2:
https://tier4.github.io/scenario_simulator_v2-docs/
2. CARLA scenario runner:
https://scenario-runner.readthedocs.io/en/latest/
