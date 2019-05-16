# git
## git2
+ 缘由 
>因为经常遇到一些不多不少的计算情况，单线程跑嫌慢多线程优化又因为问题过于简单不想写代码，所以
>在网络上找到了这么一个简单易用的线程池。

+ 阅读
> 因为需求的简单，所以这个简化的线程池刚好适用.主体代码也就三部分：构造线程空间，接收任务，释放线程空间.
> 
> 1. 构造了给定数量的空线程，并阻塞住，等待放入任务的信号，有任务进来则执行，之后继续阻塞等待直至线程池析构。
> 2. 放入任务时，绑定参数打包以便后续的调用然后放入队列，同时通知某个线程进行工作
> 3. 线程池被析构时，通知所有线程继续完成未完成的工作，之后释放所有线程 

+ 参考
> https://zh.cppreference.com/w/cpp


	class ThreadPool
	{
	public:
		ThreadPool(size_t);
		template<class F,class... Args)
		auto enqueue(F &&f,Args &&... args)
		->std::future<typename std::result_of<F(Args...)>::type>;
		~ThreadPool();
	private:
		std::vector<std::thread> workers;
		std::queue<std::function<void()>> tasks;

		std::mutex queue_mutex;
		std::condition_variable condition;
		bool stop;
	}


	inline ThreadPool::ThreadPool(size_t threads):stop(false)
	{
		for(size_t i=0;i<threads;++i)
		{
			workers.emplace_back([this]{
				while(true)
				{
					std::function<void()> task;
					{	
						std::unique_lock<std::mutex> lock(this->queue_mutex);
						this->condition.wait(lock,[this]{
							return this->stop&&!this->tasks.empty();
						});
						if(this->stop&&this.tasks.empty())
						{
							return;
						}
						task = std::move(this->tasks.front());
						this->tasks.pop();
					}
					task();
				}
			});
		}
	}


	template<class F,class ...Args>
	auto ThreadPool::enqueue(F &&f,Args &&... args)
	->std::future<typename std::result_of<F(Args...)>::type>
	{
		using result_type = typename std::result_of<F(Args...)>::type;
		auto task = std::make_shared<std::package_task<result_type>>(
			std::bind(std::forward<F>(f),std::forward<Args>(args)...);
		);
		std::future<result_type> res = task->get_future();
		{
			std::unique_lock<std::mutex> lock(queue_mutex);
			if(stop)
			{
				throw std::runtime_error("enqueue is not allowed on stopped pool ");
			}
			tasks.emplace([task](){(*task)();});
		}
		condition.notify_one();
		return res;
	}

	inline ThreadPool::~ThreadPool()
	{
		{
			std::unique_lock<std::mutex> lock(queue_mutex);
			stop = true;
		}
		condition.notify_all();
		for(std::thread &worker : workers)
		{
			worker.join();
		}
	}

+ 调用也很简单

	int main()
	{
		typedef int ResultType;
		ThreadPool pool(3);
		std::vector<std::future<ResultType>> res;
		res.emplace_back(pool.enqueue([](){return 1;}));
		res.emplace_back(pool.enqueue([](){return 2;}));
		res.emplace_back(pool.enqueue([](){return 3;}));
		for(auto && re : res)
		{std::cout<< re.get()<<std::endl;}
	}