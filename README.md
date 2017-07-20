# Stanford OpenSim RL NIPS2017

you know, to walk the skeleton.

original DDPG code from <https://github.com/ctmakro/gymnastics>

more details at <https://github.com/stanfordnmbl/osim-rl>

# Dependencies

  - Python 3.5
  - TF
  - matplotlib
  - pymsgbox (act as a stop button instead of using ctrl-c)
  - Canton
  - numpy, gym
  - osim-rl
  - OpenCV3

# To Run

```bash
$ ipython -i ddpg2.py
```

then enter `r(100)` to train the agent for 100 episodes.

# Note for users on Win7 x64 + Python 3.5 (2017-07-06)

Assume you want to run osim-rl on Windows w/py35, since TensorFlow support only Python 3.5 on Windows.

1. Build OpenSim yourself (since @kidzik didn't do this for us)
    - Install VC++ 2015 build tools. Should not take long

    - CMD
        - Clone the opensim conda builder, then run the build command:

            ```bash
            (chinese users) > set "HTTPS_PROXY=whatever_you_use"
            > git clone https://github.com/opensim-org/conda-opensim.git
            > cd conda-opensim
            > conda build opensim --python 3.5
            ```

            Above will fail on Chinese Windows due to file encoding error. If so, don't run `conda build` again, as that will create a new build environment and pull the whole OpenSim from GitHub again.

            Instead, open `D:\Anaconda3\conda-bld\opensim_1499279773305\work\dependencies\BTK\Utilities\Open3DMotion\src\Open3DMotion\MotionFile\Formats\MDF\FileFormatMDF.cpp` and replace every occurence of the square symbol (superscript "2") with "^2".

            The file should now look like:
            ```c
            //...
            { "Marker Acceleration", "m/s^2", "%.3fm/s2", "%.3f", (float)0.05},
            //...
            ```

        - Run the build script generated by the build process above again:

            ```bash
            > cd D:\Anaconda3\conda-bld\opensim_1499279773305(actual-path-may-vary)\work
            > bld.bat
            ```

            The compilation should run smoothly this time.

        - Since CMD was used to run `bld.bat`, you may notice that some commands, such as an `cp` (copy) did not run successfully. It should produce errors like that:

            ```bash
            (d:\Anaconda3\conda-bld\opensim_1499279773305\_b_env) Nonecp d:\Anaconda3\conda-bld\opensim_1499279773305\_b_env\Library
            \simbody\bin\simbody-visualizer.exe   d:\Anaconda3\conda-bld\opensim_1499279773305\_b_env\simbody-visualizer.exe
            'cp' is not recognized as an internal or external command,
            operable program or batch file.`
            ```

            you can do the copy manually, though.

        - also the `python setup.py install` might not run smoothly:

            ```bash
            (d:\Anaconda3\conda-bld\opensim_1499279773305\_b_env) Nonepython setup.py install
            Traceback (most recent call last):
              File "setup.py", line 8, in <module>
                execfile('opensim/version.py')
            NameError: name 'execfile' is not defined
            ```

            - cause

              `setup.py` is written in py2, so it seems that i built the py2(default) branch of OpenSim.

            - solution

              in `setup.py`, replace `execfile(name)` with `exec(open(name).read())`.

            Then `cd` to the correct path (`D:\Anaconda3\conda-bld\opensim_1499279773305\_b_env\Library\sdk\python`) and manually run `pip install -e .`

      - Now test the OpenSim installation:
          - try `import opensim` in python.

            upon `import opensim` python may fail to find simbody's libraries.

          - solution

            add `D:\Anaconda3\conda-bld\opensim_1499279773305\work\opensim_build\Release` to your PATH.

2. install osim-rl
    - CMD

        ```bash
        (chinese users) > set HTTPS_PROXY=whatever_you_use
        > git clone https://github.com/stanfordnmbl/osim-rl.git
        > cd osim-rl
        > pip install -e .
        > cd tests
        > python test.manager.py
        ```

        You should now see the skeleton swinging!

# The simulation is too slow
- modify `D:\Anaconda3\conda-bld\opensim_1499279773305\work\OpenSim\Simulation\Manager\Manager.cpp` as follows:

    ```c
    Manager::Manager(Model& model) : Manager(model, true)
    {
        SimTK::Integrator *vi = new SimTK::RungeKutta2Integrator(_model->getMultibodySystem());
        vi->setAccuracy(3e-2); // reduce accuracy saves time
        _defaultInteg.reset(vi);
        _integ = *_defaultInteg;
    }
    ```

    then build the whole thing again by running `bld.bat` mentioned above.

- parallelize the training environment (see code)
