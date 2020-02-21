
# Benchmark Configuration

In this section you will learn

1. The `OptaPlanner Benchmark` configuration file.

2. How to configure multiple solver configurations for a benchmark run.

3. How to run a benchmark from the configuration file.

In the previous lab we've implemented our a `PlannerBenchmark` from an existing solver configuration. This allowed us to quickly setup and run a basic benchmark. In this lab we will create a benchmark configuration file, in which we can define multiple solver configurations, which allows us to compare multiple algorithms and configurations.

## The Benchmark Config

The OptaPlanner Benchmark Configuration is defined in an XML file. In this file we can configure:

* datasets
* solver configurations, including algorithm configurations.

This allows to test different solves with different datasets, publish their results in a HTML report, and compare them. The report gives us insight in the performance of different algorithms different datasets, as well as the performance of our solvers as a whole. This makes it possible to, for example, identify performance problems, identify solvers that require coarser moves, etc. In general, it's a perfect way to assess the health of your OptaPlanner solution, and pick the right solver configuration for your specific problem.

1. Open CodeReady Workspaces and open your project from the previous lab. You can also import the following project from GitHub and use it as a starting point for this module: [https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m3-labs-step-2](https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m1-labs-step-2)

2. Create a new folder called `org/optaplanner/examples/cloudbalancing/benchmark` in the `src/main/resources` folder.

3. In this folder, create a new file called `cloudBalancingBenchmarkConfig.xml`. Add the following XML configuration to this file:

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<plannerBenchmark>
  <benchmarkDirectory>local/benchmarkReport</benchmarkDirectory>
  <parallelBenchmarkCount>AUTO</parallelBenchmarkCount>

  <inheritedSolverBenchmark>
    <problemBenchmarks>
      <xStreamAnnotatedClass>org.optaplanner.examples.cloudbalancing.domain.CloudBalance</xStreamAnnotatedClass>
      <inputSolutionFile>data/cloudbalancing/unsolved/4computers-12processes.xml</inputSolutionFile>
      <inputSolutionFile>data/cloudbalancing/unsolved/100computers-300processes.xml</inputSolutionFile>
    </problemBenchmarks>

    <solver>
      <scanAnnotatedClasses/>
      <scoreDirectorFactory>
        <scoreDrl>org/optaplanner/examples/cloudbalancing/solver/cloudBalancingScoreRules.drl</scoreDrl>
        <initializingScoreTrend>ONLY_DOWN/ONLY_DOWN</initializingScoreTrend>
      </scoreDirectorFactory>
      <termination>
        <millisecondsSpentLimit>40000</millisecondsSpentLimit>
      </termination>
    </solver>
  </inheritedSolverBenchmark>

  <solverBenchmark>
    <name>Late Acceptance</name>
    <solver>
      <constructionHeuristic>
        <constructionHeuristicType>FIRST_FIT</constructionHeuristicType>
      </constructionHeuristic>
      <localSearch>
        <localSearchType>LATE_ACCEPTANCE</localSearchType>
      </localSearch>
    </solver>
  </solverBenchmark>
  <solverBenchmark>
    <name>Tabu Search</name>
    <solver>
      <constructionHeuristic>
        <constructionHeuristicType>FIRST_FIT</constructionHeuristicType>
      </constructionHeuristic>
      <localSearch>
        <localSearchType>TABU_SEARCH</localSearchType>
      </localSearch>
    </solver>
  </solverBenchmark>
</plannerBenchmark>
~~~

4. Open the `CloudBalancingBenchmarker.java` class we created in the previous lab. Replace the content of the `main` method with the following code. We now create the `PlannerBenchmark` from our new configuration file, instead of an existing solver config. Also, we no longer need to programmatically load our datasets, as these our now configured in our benchmark configuration file.

~~~java
public static void main(String[] args) {
  PlannerBenchmarkFactory benchmarkFactory = PlannerBenchmarkFactory.createFromXmlResource("org/optaplanner/examples/cloudbalancing/benchmark/cloudBalancingBenchmarkConfig.xml");

  PlannerBenchmark benchmark = benchmarkFactory.buildPlannerBenchmark();
  benchmark.benchmark();
}
~~~

4. Save the file (note that in CodeReady Workspaces, files are automatically saved).

Like in the previous lab, we can now run the benchmark using our Maven profile:

~~~
mvn clean install exec:java -DskipTests -Pbenchmarker -f ${current.project.path}
~~~

Note that we've set the `termination` configuration in our benchmark configuration file to 40 seconds. Also, we will be running 4 benchmark runs, one for each _local search algorithm_-dataset combination, so, running this benchmark can take some time.

After the benchmark has completed, a benchmark report will be available in the `local/benchmarkReport/{date-time}` folder in your CodeReady workspace.

1. Open the `index.html` file in the `local/benchmarkReport/{date-time}` folder. You can open the `index.html` by right-clicking on the file and selecting _"Preview"_.

image:images/codeready-benchmarker-index-html.png[]

2. Note that this benchmark reports contains.

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
