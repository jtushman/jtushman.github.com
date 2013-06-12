Sometimes a heroku dyno is too constrained to perform certain tasks.  But I have grown quite accustomed to their elegant CLI.  So here I outline a proposed API for managing workers outside the heroku world.

I think it will be a fabric extension.


## API

1.  List workers and their state

		python aws_scaler.py workers ps

2.  Scale worker to n
	
		fab scale:workers=n
		
3. Workers logs

		fab aws_scaler.py workers logs
		
3. Tail Workers logs

		fab aws_scaler.py workers logs:tail=true
		
1. Deploy latest code base

		fab aws_scaler.py workers deploy
	

## Implementation Notes

I think most of the implementation is pretty plain jane, using `boto`  Execept for the `deploy` command.  This is my current thoughts on implementation

### Prerequisites

1. You have set up a prototype AMI that run your code base.  You can use this [one](http://) which a ubuntu 12.04 box with python 2.7.3, and uses upstart for manage processes.

2. You have ProcFile [a la heroku](http://) defining the deamon process that you want the worker to run.  For example:

		worker: FLASK_ENV=production python run_worker.py

note: do not handle logging in your procfile.  aws_scaler will handle that for you


### Deployment workflow
1. User runs  `fab aws_scaler.py workers deploy`
2. We create a new instance based on the Prototype
3. Update the OS
3. Using `git` deploy the application to /apps/[app_name]
4. In the application directory run `pip install -r requirements.txt`
5. Copy the `worker` line into a upstart init template, that also `tees` the output to /var/worker.log
6. Upon success save the new state of the AMI to [app_name].v[version_number]
7. Shut down the new prototype
8. Shut down running workers
9. Start n number of new worker based on the new worker
 