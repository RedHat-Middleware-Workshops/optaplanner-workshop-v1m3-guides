
# Benchmark

In this section you will learn

1. What the  `OptaPlanner Benchmark` is.

2. The different _Heuristic Algorithms_ that OptaPlanner supports.

3. How to choose the right algorithm for your problem.

In the previous lab we've implemented our `ScoreCalculator` and _constraints_ and we ran our first full run of our optimization application for the _Cloud Balancing_ problem. Let's now try to power-tweak our application by configuring different heuristic algorithms.

## The Benchmark

OptaPlanner Benchmarker can be used to benchmark:

* different algorithms.
* different algorithm configurations.
* different datasets.

and a combination of the above. It's an excellent tool to benchmark your solution and to create a report that indicates if your OptaPlanner solution performs as expected. It also helps to _power tweak_ your solution to create even better solutions to your problem than you already have (notice that a 2% better score can mean a substantial amount of resources and/or money in a real world application).


1. Open CodeReady Workspaces and open your project from the previous lab. You can also import the following project from GitHub and use it as a starting point for this module: [https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m3-labs-step-1](https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m1-labs-step-1)

2. Create a new package called `org.optaplanner.examples.cloudbalancing.benchmark` in the `src/main/java` folder. We can do this in CodeReady Workspaces by creating a new folder called `org/optaplanner/examples/cloudbalancing/benchmark` in the `src/main/java` folder.

3. In this folder, create a new file called `CloudBalancingBenchmarker.java`.

4. Create a `main` method:

~~~java
public static void main(String[] args) {
}
~~~

5. Save the file (note that in CodeReady Workspaces, files are automatically saved).

OptaPlanner provides a very simple, easy to use, API that gets you started with the OptaPlanner Benchmark in just a few lines of code. On top of that, OptaPlanner provides a vast array of configuration options to configure and tweak the Benchmarker to your specific needs.

The easiest way to get started is to create a Benchmarker from an existing `SolverFactory`, like the `SolverFactory` we create in our `CloudBalancingSolverTest`.

1. In the `main` method of our `CloudBalancingBenchmarker` class, add the following code:

~~~java
public static void main(String[] args) {
  SolverFactory<CloudBalance> solverFactory = SolverFactory.createFromXmlResource("org/optaplanner/examples/cloudbalancing/solver/cloudBalancingSolverConfig.xml");
  PlannerBenchmarkFactory benchmarkFactory = PlannerBenchmarkFactory.createFromSolverFactory(solverFactory);
}
~~~

2. If the required imports are not automatically added, use the CodeReadyWorkspaces _Organize Imports_ functionality to add the imports.

3. Save the file (note that in CodeReady Workspaces, files are automatically saved).

This creates a new `BenchmarkFactory` component from an existing `SolverFactory`. It uses the _Solver Configuration_ configured on the `SolverFactory`. Hence, the Benchmark will run with the exact same configuration as the configuration we've used in our unit-tests so far.

Next, we need to load the data-set we want to use in our benchmark. In this case we will use the data-set that contains 100 computers and 300 processes. Note that we can re-user our `CloudBalanceRepository` class to load the data-set.

1. In the `main` method of our `CloudBalancingBenchmark`, add the code to load the dataset.

~~~java
public static void main(String[] args) {
  SolverFactory<CloudBalance> solverFactory = SolverFactory.createFromXmlResource("org/optaplanner/examples/cloudbalancing/solver/cloudBalancingSolverConfig.xml");
  PlannerBenchmarkFactory benchmarkFactory = PlannerBenchmarkFactory.createFromSolverFactory(solverFactory);

  //Project build directory is provided as an argument to this method
  String projectBuildPath = args[0];
  File dataSetOne = new File(projectBuildPath + File.separator + "data/cloudbalancing/unsolved/100computers-300processes.xml");
  CloudBalance cloudBalanceDataSet1 = CloudBalanceRepository.load(dataSetOne);
}
~~~

2. With our data-set loaded, we can now create and run the Benchmarker.

~~~java
public static void main(String[] args) {
  SolverFactory<CloudBalance> solverFactory = SolverFactory.createFromXmlResource("org/optaplanner/examples/cloudbalancing/solver/cloudBalancingSolverConfig.xml");
  PlannerBenchmarkFactory benchmarkFactory = PlannerBenchmarkFactory.createFromSolverFactory(solverFactory);

  //Project build directory is provided as an argument to this method
  String projectBuildPath = args[0];
  File dataSetOne = new File(projectBuildPath + File.separator + "data/cloudbalancing/unsolved/100computers-300processes.xml");
  CloudBalance cloudBalanceDataSet1 = CloudBalanceRepository.load(dataSetOne);

  PlannerBenchmark benchmark = benchmarkFactory.buildPlannerBenchmark(cloudBalanceDataSet1);
  benchmark.benchmark();
}
~~~

3. Save the file (note that in CodeReady Workspaces, files are automatically saved).

We can now configure our project's Maven `pom.xml` file and add a profile that allows us to run our `Benchmark` class from Maven.

1. Add the following `profile` configuration to your project's `pom.xml` file, directly under the closing `</dependencies>` tag:

~~~xml
<profiles>
  <profile>
    <id>benchmarker</id>
    <build>
      <plugins>
        <plugin>
          <groupId>org.codehaus.mojo</groupId>
          <artifactId>exec-maven-plugin</artifactId>
          <version>1.6.0</version>
          <executions>
            <execution>
              <goals>
                <goal>java</goal>
              </goals>
            </execution>
          </executions>
          <configuration>
            <mainClass>org.optaplanner.examples.cloudbalancing.benchmark.CloudBalancingBenchmarker</mainClass>
            <arguments>
              <argument>${project.basedir}</argument>
            </arguments>
          </configuration>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
~~~

2. Save the `pom.xml` file.


We now need to execute the class. The Maven _profile_ we've just added to the `pom.xml` file executes the `CloudBalancingBenchmarker` class. To execute the Maven goal with the given profile, we need to create a new Maven build script.

1. Open the _"Manage commands"_ section in your CodeReady Workspaces environment:

image:images/codeready-manage-commands.png[]


2. In the _"Build"_ category, click the `+` button to add a new _build configuration_. Select _"Maven"_.

image:images/codeready-benchmarker-maven-build.png[]

3. Name your new build configuration `benchmarker` and configure it with the following line:

~~~
mvn clean install exec:java -DskipTests -Pbenchmarker -f ${current.project.path}
~~~

image:images/codeready-benchmarker-maven-build-config.png[]

4. Run the build by clicking on the green _"Run"_ button.

After the benchmark has completed, a benchmark report will be available in the `local/benchmarkReport/{date-time}` folder in your CodeReady workspace.

1. Open the `index.html` file in the `local/benchmarkReport/{date-time}` folder. You can open the `index.html` by right-clicking on the file and selecting _"Preview"_.

image:images/codeready-benchmarker-index-html.png[]

2. Inspect the _Best Score Summary_ section:
    * This section visualizes which _solver configuration_ produced the best score. Since we've only run the benchmarker with a single _solver configuration_. This graph currently only shows one entry. This graph is useful when you've defined multiple _solver configurations_, using different heuristic algorithmns, or different configurations of these algorithms, and you want to analyse which configuration works best for your problem and data-set.

image:images/benchmark-best-score-summary.png[]

3. Inspect the _Performance Summary_:
    * This section shows the _Score Calculation Speed* of your configuration. This calculation speed is extremely important in an OptaPlanner project. The higher the calculation speed, the more moves OptaPlanner can execute and calculate in a given timeframe, the better the solution that OptaPlanner provides. A _Score Calculation Speed_ below 1000m should be addressed.

image:images/benchmark-performance-summary.png[]

4. Inspect the _Problem Benchmarks_ section:
    * This part of the report shows the progression of the _score_ over _time. As stated earlier, in the beginning of an OptaPlanner run, the score usually improves very quickly, after which finding better solutions slows down. The graph shown in the benchmark report clearly visualizes this. This is a great tool to:
        ... Inspect whether your solver follows this common pattern. If it does not, this usually indicates a problem in your solution, for example, a _score calculation count_ that is too low, or a problem that is stuck in a _local optima_.
        ... Compare the score progression of multiple solver configurations to determine which solver configuration works best for your planning problem.
        ... The point of _diminishing returns_, which can indicate how long you should run the solver (on an environment with comparable hardware).

image:images/optaplanner-benchmark.png[]


Furthermore, the report shows the actual _Solver Configuration_ used for each run, as well as information of the system on which the benchmark ran.

You've successfully implemented your first OptaPlanner Benchmarker. In the next lab, we
