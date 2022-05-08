# How to Handle Periodic Tasks in Django with Celery ( Dockerized )


## Run:

```sh
$ docker-compose up -d --build
```
Check logs:

```sh
$ docker-compose logs -f 'celery'
```

## Explanation:


<html lang="en">

<body class=" page-blog page-blog-detail">

<main>
  <div class="container blog-container" style="padding-top: 0;">
    <div class="row">
      <div class="col col-12 col-lg-8">
        


<p>Since we'll need to manage four processes in total (Django, Redis, worker, and scheduler), we'll use Docker to simplify our workflow by wiring them up so that they can all be run from one terminal window with a single command.</p>
<p>From the project root, create the images and spin up the Docker containers:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose up -d --build
</code></pre></div>

<p>Next, apply the migrations:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose <span class="nb">exec</span> web python manage.py migrate
</code></pre></div>

<p>Once the build is complete, navigate to <a href="http://localhost:1337">http://localhost:1337</a> to ensure the app works as expected. You should see the following text:</p>
<div class="codehilite"><pre><span></span><code>Orders

No orders found!
</code></pre></div>

<p>Take a quick look at the project structure before moving on:</p>
<div class="codehilite"><pre><span></span><code>├── .gitignore
├── docker-compose.yml
└── project
    ├── Dockerfile
    ├── core
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── entrypoint.sh
    ├── manage.py
    ├── orders
    │   ├── __init__.py
    │   ├── admin.py
    │   ├── apps.py
    │   ├── migrations
    │   │   ├── 0001_initial.py
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   ├── urls.py
    │   └── views.py
    ├── products.json
    ├── requirements.txt
    └── templates
        └── orders
            └── order_list.html
</code></pre></div>


<h2 id="celery-and-redis">Celery and Redis</h2>
<p>Now, we need to add containers for Celery, Celery Beat, and Redis.</p>
<p>We'll begin by adding the dependencies to the <em>requirements.txt</em> file:</p>
<div class="codehilite"><pre><span></span><code>Django==3.2.4
celery==5.1.2
redis==3.5.3
</code></pre></div>

<p>Next, add the following to the end of the <em>docker-compose.yml</em> file:</p>
<div class="codehilite"><pre><span></span><code><span class="nt">redis</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">image</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis:alpine</span><span class="w"></span>
<span class="nt">celery</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">  </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">celery -A core worker -l info</span><span class="w"></span>
<span class="w">  </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">  </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="8ebdfcb7a3b3f4a3a5d1e3f4bafca3afffebebeace">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="432403287b292c7b3a70317174">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">  </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"></span>
<span class="nt">celery-beat</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">  </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">celery -A core beat -l info</span><span class="w"></span>
<span class="w">  </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">  </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="ebd899d2c6d691c6c0b48691df99c6ca9a8e8e8fab">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="583f183360323760216b2a6a6f">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">  </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"></span>
</code></pre></div>

<p>We also need to update the web service's <code>depends_on</code> section:</p>
<div class="codehilite"><pre><span></span><code><span class="nt">web</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">  </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">python manage.py runserver 0.0.0.0:8000</span><span class="w"></span>
<span class="w">  </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">  </span><span class="nt">ports</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">1337:8000</span><span class="w"></span>
<span class="w">  </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="b487c68d9989ce999febd9ce80c69995c5d1d1d0f4">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="d2b592b9eab8bdeaabe1a0e0e5">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">  </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"> </span><span class="c1"># NEW</span><span class="w"></span>
</code></pre></div>

<p>The full <em>docker-compose.yml</em> file should now look like this:</p>
<div class="codehilite"><pre><span></span><code><span class="nt">version</span><span class="p">:</span><span class="w"> </span><span class="s">&#39;3.8&#39;</span><span class="w"></span>

<span class="nt">services</span><span class="p">:</span><span class="w"></span>
<span class="w">  </span><span class="nt">web</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">    </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">python manage.py runserver 0.0.0.0:8000</span><span class="w"></span>
<span class="w">    </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">    </span><span class="nt">ports</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">1337:8000</span><span class="w"></span>
<span class="w">    </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="3407460d19094e191f6b594e004619154551515074">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="224562491a484d1a5b11501015">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">    </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"></span>
<span class="w">  </span><span class="nt">redis</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="nt">image</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis:alpine</span><span class="w"></span>
<span class="w">  </span><span class="nt">celery</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">    </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">celery -A core worker -l info</span><span class="w"></span>
<span class="w">    </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">    </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="2c1f5e150111560107734156185e010d5d4949486c">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="a7c0e7cc9fcdc89fde94d59590">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">    </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"></span>
<span class="w">  </span><span class="nt">celery-beat</span><span class="p">:</span><span class="w"></span>
<span class="w">    </span><span class="nt">build</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project</span><span class="w"></span>
<span class="w">    </span><span class="nt">command</span><span class="p">:</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">celery -A core beat -l info</span><span class="w"></span>
<span class="w">    </span><span class="nt">volumes</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">./project/:/usr/src/app/</span><span class="w"></span>
<span class="w">    </span><span class="nt">environment</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DEBUG=1</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">SECRET_KEY=dbaa1_i7%*<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="4271307b6f7f386f691d2f3876306f633327272602">[email&#160;protected]</a>(-a_r(<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="6e092e0556040156175d1c5c59">[email&#160;protected]</a>%m</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]</span><span class="w"></span>
<span class="w">    </span><span class="nt">depends_on</span><span class="p">:</span><span class="w"></span>
<span class="w">      </span><span class="p p-Indicator">-</span><span class="w"> </span><span class="l l-Scalar l-Scalar-Plain">redis</span><span class="w"></span>
</code></pre></div>

<p>Before building the new containers we need to configure Celery in our Django app.</p>
<h2 id="celery-configuration">Celery Configuration</h2>
<h3 id="setup">Setup</h3>
<p>In the "core" directory, create a <em>celery.py</em> file and add the following code:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">import</span> <span class="nn">os</span>

<span class="kn">from</span> <span class="nn">celery</span> <span class="kn">import</span> <span class="n">Celery</span>


<span class="n">os</span><span class="o">.</span><span class="n">environ</span><span class="o">.</span><span class="n">setdefault</span><span class="p">(</span><span class="s2">&quot;DJANGO_SETTINGS_MODULE&quot;</span><span class="p">,</span> <span class="s2">&quot;core.settings&quot;</span><span class="p">)</span>

<span class="n">app</span> <span class="o">=</span> <span class="n">Celery</span><span class="p">(</span><span class="s2">&quot;core&quot;</span><span class="p">)</span>
<span class="n">app</span><span class="o">.</span><span class="n">config_from_object</span><span class="p">(</span><span class="s2">&quot;django.conf:settings&quot;</span><span class="p">,</span> <span class="n">namespace</span><span class="o">=</span><span class="s2">&quot;CELERY&quot;</span><span class="p">)</span>
<span class="n">app</span><span class="o">.</span><span class="n">autodiscover_tasks</span><span class="p">()</span>
</code></pre></div>

<p>What's happening here?</p>
<ol>
<li>First, we set a default value for the <code>DJANGO_SETTINGS_MODULE</code> environment variable so that the Celery will know how to find the Django project.</li>
<li>Next, we created a new Celery instance, with the name <code>core</code>, and assigned the value to a variable called <code>app</code>.</li>
<li>We then loaded the celery configuration values from the settings object from <code>django.conf</code>. We used <code>namespace="CELERY"</code> to prevent clashes with other Django settings. All config settings for Celery must be prefixed with <code>CELERY_</code>, in other words.</li>
<li>Finally, <code>app.autodiscover_tasks()</code> tells Celery to look for Celery tasks from applications defined in <code>settings.INSTALLED_APPS</code>.</li>
</ol>
<p>Add the following code to <em>core/__init__.py</em>:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">.celery</span> <span class="kn">import</span> <span class="n">app</span> <span class="k">as</span> <span class="n">celery_app</span>

<span class="n">__all__</span> <span class="o">=</span> <span class="p">(</span><span class="s2">&quot;celery_app&quot;</span><span class="p">,)</span>
</code></pre></div>

<p>Lastly, update the <em>core/settings.py</em> file with the following Celery settings so that it can connect to Redis:</p>
<div class="codehilite"><pre><span></span><code><span class="n">CELERY_BROKER_URL</span> <span class="o">=</span> <span class="s2">&quot;redis://redis:6379&quot;</span>
<span class="n">CELERY_RESULT_BACKEND</span> <span class="o">=</span> <span class="s2">&quot;redis://redis:6379&quot;</span>
</code></pre></div>

<p>Build the new containers to ensure that everything works:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose up -d --build
</code></pre></div>

<p>Take a look at the logs for each service to see that they are ready, without errors:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose logs <span class="s1">&#39;web&#39;</span>
$ docker-compose logs <span class="s1">&#39;celery&#39;</span>
$ docker-compose logs <span class="s1">&#39;celery-beat&#39;</span>
$ docker-compose logs <span class="s1">&#39;redis&#39;</span>
</code></pre></div>

<p>If all went well, we now have four containers, each with different services.</p>
<p>Now we're ready to create a sample task to see that it works as it should.</p>
<h3 id="create-a-task">Create a Task</h3>
<p>Create a new file called <em>core/tasks.py</em> and add the following code for a sample task that just logs to the console:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">celery</span> <span class="kn">import</span> <span class="n">shared_task</span>
<span class="kn">from</span> <span class="nn">celery.utils.log</span> <span class="kn">import</span> <span class="n">get_task_logger</span>


<span class="n">logger</span> <span class="o">=</span> <span class="n">get_task_logger</span><span class="p">(</span><span class="vm">__name__</span><span class="p">)</span>


<span class="nd">@shared_task</span>
<span class="k">def</span> <span class="nf">sample_task</span><span class="p">():</span>
    <span class="n">logger</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s2">&quot;The sample task just ran.&quot;</span><span class="p">)</span>
</code></pre></div>

<h3 id="schedule-the-task">Schedule the Task</h3>
<p>At the end of your <em>settings.py</em> file, add the following code to schedule <code>sample_task</code> to run once per minute, using Celery Beat:</p>
<div class="codehilite"><pre><span></span><code><span class="n">CELERY_BEAT_SCHEDULE</span> <span class="o">=</span> <span class="p">{</span>
    <span class="s2">&quot;sample_task&quot;</span><span class="p">:</span> <span class="p">{</span>
        <span class="s2">&quot;task&quot;</span><span class="p">:</span> <span class="s2">&quot;core.tasks.sample_task&quot;</span><span class="p">,</span>
        <span class="s2">&quot;schedule&quot;</span><span class="p">:</span> <span class="n">crontab</span><span class="p">(</span><span class="n">minute</span><span class="o">=</span><span class="s2">&quot;*/1&quot;</span><span class="p">),</span>
    <span class="p">},</span>
<span class="p">}</span>
</code></pre></div>

<p>Here, we defined a periodic task using the <a href="https://docs.celeryq.dev/en/latest/userguide/configuration.html#std:setting-beat_schedule">CELERY_BEAT_SCHEDULE</a> setting. We gave the task a name, <code>sample_task</code>, and then declared two settings:</p>
<ol>
<li><code>task</code> declares which task to run.</li>
<li><code>schedule</code> sets the interval on which the task should run. This can be an integer, a timedelta, or a crontab. We used a crontab pattern for our task to tell it to run once every minute. You can find more info on Celery's scheduling <a href="https://docs.celeryq.dev/en/stable/reference/celery.schedules.html">here</a>.</li>
</ol>
<p>Make sure to add the imports:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">celery.schedules</span> <span class="kn">import</span> <span class="n">crontab</span>

<span class="kn">import</span> <span class="nn">core.tasks</span>
</code></pre></div>

<p>Restart the container to pull in the new settings:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose up -d --build
</code></pre></div>

<p>Once done, take a look at the celery logs in the container:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose logs -f <span class="s1">&#39;celery&#39;</span>
</code></pre></div>

<p>You should see something similar to:</p>
<div class="codehilite"><pre><span></span><code>celery_1  <span class="p">|</span>  -------------- <span class="o">[</span>queues<span class="o">]</span>
celery_1  <span class="p">|</span>                 .&gt; celery           <span class="nv">exchange</span><span class="o">=</span>celery<span class="o">(</span>direct<span class="o">)</span> <span class="nv">key</span><span class="o">=</span>celery
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span> <span class="o">[</span>tasks<span class="o">]</span>
celery_1  <span class="p">|</span>   . core.tasks.sample_task
</code></pre></div>

<p>We can see that Celery picked up our sample task, <code>core.tasks.sample_task</code>.</p>
<p>Every minute you should see a row in the log that ends with "The sample task just ran.":</p>
<div class="codehilite"><pre><span></span><code>celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">03</span>:06:00,003: INFO/MainProcess<span class="o">]</span>
              Task core.tasks.sample_task<span class="o">[</span>b8041b6c-bf9b-47ce-ab00-c37c1e837bc7<span class="o">]</span> received
celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">03</span>:06:00,004: INFO/ForkPoolWorker-8<span class="o">]</span>
              core.tasks.sample_task<span class="o">[</span>b8041b6c-bf9b-47ce-ab00-c37c1e837bc7<span class="o">]</span>:
              The sample task just ran.
</code></pre></div>

<h2 id="custom-django-admin-command">Custom Django Admin Command</h2>
<p>Django provides a <a href="https://docs.djangoproject.com/en/3.2/ref/django-admin/#available-commands">number</a> of built-in <code>django-admin</code> commands, like:</p>
<ul>
<li><code>migrate</code></li>
<li><code>startproject</code></li>
<li><code>startapp</code></li>
<li><code>dumpdata</code></li>
<li><code>makemigrations</code></li>
</ul>
<p>Along with the built-in commands, Django also gives us the option to create our own <a href="https://docs.djangoproject.com/en/3.2/howto/custom-management-commands/">custom commands</a>:</p>

<p>So, we'll first configure a new command and then use Celery Beat to run it automatically.</p>
<p>Start by creating a new file called <em>orders/management/commands/my_custom_command.py</em>. Then, add the minimal required code for it to run:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">django.core.management.base</span> <span class="kn">import</span> <span class="n">BaseCommand</span><span class="p">,</span> <span class="n">CommandError</span>


<span class="k">class</span> <span class="nc">Command</span><span class="p">(</span><span class="n">BaseCommand</span><span class="p">):</span>
    <span class="n">help</span> <span class="o">=</span> <span class="s2">&quot;A description of the command&quot;</span>

<span class="k">def</span> <span class="nf">handle</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">options</span><span class="p">):</span>
        <span class="k">pass</span>
</code></pre></div>

<p>The <code>BaseCommand</code> has a few <a href="https://docs.djangoproject.com/en/3.2/howto/custom-management-commands/#methods">methods</a> that can be overridden, but the only method that's required is <code>handle</code>. <code>handle</code> is the entry point for custom commands. In other words, when we run the command, this method is called.</p>
<p>To test, we'd normally just add a quick print statement. However, it's recommended to use <code>stdout.write</code> instead per the Django documentation:</p>

<p>So, add a <code>self.stdout.write</code> command:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">django.core.management.base</span> <span class="kn">import</span> <span class="n">BaseCommand</span><span class="p">,</span> <span class="n">CommandError</span>


<span class="k">class</span> <span class="nc">Command</span><span class="p">(</span><span class="n">BaseCommand</span><span class="p">):</span>
    <span class="n">help</span> <span class="o">=</span> <span class="s2">&quot;A description of the command&quot;</span>

<span class="k">def</span> <span class="nf">handle</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">options</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">stdout</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="s2">&quot;My sample command just ran.&quot;</span><span class="p">)</span> <span class="c1"># NEW</span>
</code></pre></div>

<p>To test, from the command line, run:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose <span class="nb">exec</span> web python manage.py my_custom_command
</code></pre></div>

<p>You should see:</p>
<div class="codehilite"><pre><span></span><code>My sample <span class="nb">command</span> just ran.
</code></pre></div>

<p>With that, let's tie everything together!</p>
<h2 id="schedule-a-custom-command-with-celery-beat">Schedule a Custom Command with Celery Beat</h2>
<p>Now that we've spun up the containers, tested that we can schedule a task to run periodically, and wrote a custom Django Admin sample command, it's time to configure Celery Beat to run the custom command periodically.</p>
<h3 id="setup_1">Setup</h3>
<p>In the project we have a very basic app called orders. It contains two models, <code>Product</code> and <code>Order</code>. Let's create a custom command that sends an email report of the confirmed orders from the day.</p>
<p>To begin with, we'll add a few products and orders to the database via the fixture included in this project:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose <span class="nb">exec</span> web python manage.py loaddata products.json
</code></pre></div>

<p>Next, add some sample orders via the Django Admin interface. To do so, first create a superuser:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose <span class="nb">exec</span> web python manage.py createsuperuser
</code></pre></div>

<p>Fill in username, email, and password when prompted. Then navigate to <a href="http://127.0.0.1:1337/admin">http://127.0.0.1:1337/admin</a> in your web browser. Log in with the superuser you just created and create a couple of orders. Make sure at least one has a <code>confirmed_date</code> of today.</p>
<p>Let's create a new custom command for our e-mail report.</p>
<p>Create a file called <em>orders/management/commands/email_report.py</em>:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">datetime</span> <span class="kn">import</span> <span class="n">timedelta</span><span class="p">,</span> <span class="n">time</span><span class="p">,</span> <span class="n">datetime</span>

<span class="kn">from</span> <span class="nn">django.core.mail</span> <span class="kn">import</span> <span class="n">mail_admins</span>
<span class="kn">from</span> <span class="nn">django.core.management</span> <span class="kn">import</span> <span class="n">BaseCommand</span>
<span class="kn">from</span> <span class="nn">django.utils</span> <span class="kn">import</span> <span class="n">timezone</span>
<span class="kn">from</span> <span class="nn">django.utils.timezone</span> <span class="kn">import</span> <span class="n">make_aware</span>

<span class="kn">from</span> <span class="nn">orders.models</span> <span class="kn">import</span> <span class="n">Order</span>

<span class="n">today</span> <span class="o">=</span> <span class="n">timezone</span><span class="o">.</span><span class="n">now</span><span class="p">()</span>
<span class="n">tomorrow</span> <span class="o">=</span> <span class="n">today</span> <span class="o">+</span> <span class="n">timedelta</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
<span class="n">today_start</span> <span class="o">=</span> <span class="n">make_aware</span><span class="p">(</span><span class="n">datetime</span><span class="o">.</span><span class="n">combine</span><span class="p">(</span><span class="n">today</span><span class="p">,</span> <span class="n">time</span><span class="p">()))</span>
<span class="n">today_end</span> <span class="o">=</span> <span class="n">make_aware</span><span class="p">(</span><span class="n">datetime</span><span class="o">.</span><span class="n">combine</span><span class="p">(</span><span class="n">tomorrow</span><span class="p">,</span> <span class="n">time</span><span class="p">()))</span>


<span class="k">class</span> <span class="nc">Command</span><span class="p">(</span><span class="n">BaseCommand</span><span class="p">):</span>
    <span class="n">help</span> <span class="o">=</span> <span class="s2">&quot;Send Today&#39;s Orders Report to Admins&quot;</span>

<span class="k">def</span> <span class="nf">handle</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="o">*</span><span class="n">args</span><span class="p">,</span> <span class="o">**</span><span class="n">options</span><span class="p">):</span>
        <span class="n">orders</span> <span class="o">=</span> <span class="n">Order</span><span class="o">.</span><span class="n">objects</span><span class="o">.</span><span class="n">filter</span><span class="p">(</span><span class="n">confirmed_date__range</span><span class="o">=</span><span class="p">(</span><span class="n">today_start</span><span class="p">,</span> <span class="n">today_end</span><span class="p">))</span>

<span class="k">if</span> <span class="n">orders</span><span class="p">:</span>
            <span class="n">message</span> <span class="o">=</span> <span class="s2">&quot;&quot;</span>

<span class="k">for</span> <span class="n">order</span> <span class="ow">in</span> <span class="n">orders</span><span class="p">:</span>
                <span class="n">message</span> <span class="o">+=</span> <span class="sa">f</span><span class="s2">&quot;</span><span class="si">{</span><span class="n">order</span><span class="si">}</span><span class="s2"> </span><span class="se">\n</span><span class="s2">&quot;</span>

<span class="n">subject</span> <span class="o">=</span> <span class="p">(</span>
                <span class="sa">f</span><span class="s2">&quot;Order Report for </span><span class="si">{</span><span class="n">today_start</span><span class="o">.</span><span class="n">strftime</span><span class="p">(</span><span class="s1">&#39;%Y-%m-</span><span class="si">%d</span><span class="s1">&#39;</span><span class="p">)</span><span class="si">}</span><span class="s2"> &quot;</span>
                <span class="sa">f</span><span class="s2">&quot;to </span><span class="si">{</span><span class="n">today_end</span><span class="o">.</span><span class="n">strftime</span><span class="p">(</span><span class="s1">&#39;%Y-%m-</span><span class="si">%d</span><span class="s1">&#39;</span><span class="p">)</span><span class="si">}</span><span class="s2">&quot;</span>
            <span class="p">)</span>

<span class="n">mail_admins</span><span class="p">(</span><span class="n">subject</span><span class="o">=</span><span class="n">subject</span><span class="p">,</span> <span class="n">message</span><span class="o">=</span><span class="n">message</span><span class="p">,</span> <span class="n">html_message</span><span class="o">=</span><span class="kc">None</span><span class="p">)</span>

<span class="bp">self</span><span class="o">.</span><span class="n">stdout</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="s2">&quot;E-mail Report was sent.&quot;</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">stdout</span><span class="o">.</span><span class="n">write</span><span class="p">(</span><span class="s2">&quot;No orders confirmed today.&quot;</span><span class="p">)</span>
</code></pre></div>

<p>In the code, we queried the database for orders with a <code>confirmed_date</code> of today, combined the orders into a single message for the email body, and used Django's built in <code>mail_admins</code> command to send the emails to the admins.</p>
<p>Add a dummy admin email and set the <code>EMAIL_BACKEND</code> to use the <a href="https://docs.djangoproject.com/en/3.2/topics/email/#console-backend">Console backend</a>, so the email is sent to stdout, in the settings file:</p>
<div class="codehilite"><pre><span></span><code><span class="n">EMAIL_BACKEND</span> <span class="o">=</span> <span class="s2">&quot;django.core.mail.backends.console.EmailBackend&quot;</span>
<span class="n">DEFAULT_FROM_EMAIL</span> <span class="o">=</span> <span class="s2">&quot;<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="432d2c3126332f3a03262e222a2f6d202c2e">[email&#160;protected]</a>&quot;</span>
<span class="n">ADMINS</span> <span class="o">=</span> <span class="p">[(</span><span class="s2">&quot;testuser&quot;</span><span class="p">,</span> <span class="s2">&quot;<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="51253422257f2422342311343c30383d7f323e3c">[email&#160;protected]</a>&quot;</span><span class="p">),</span> <span class="p">]</span>
</code></pre></div>

<p>It should now be possible to run our new command from the terminal.</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose <span class="nb">exec</span> web python manage.py email_report
</code></pre></div>

<p>And the output should look similar to this:</p>
<div class="codehilite"><pre><span></span><code>Content-Type: text/plain<span class="p">;</span> <span class="nv">charset</span><span class="o">=</span><span class="s2">&quot;utf-8&quot;</span>
MIME-Version: <span class="m">1</span>.0
Content-Transfer-Encoding: 7bit
Subject: <span class="o">[</span>Django<span class="o">]</span> Order Report <span class="k">for</span> <span class="m">2021</span>-07-01 to <span class="m">2021</span>-07-02
From: <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="1e6c71716a5e72717d7f7276716d6a">[email&#160;protected]</a>
To: <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="93e7f6e0e7bde6e0f6e1d3f6fef2faffbdf0fcfe">[email&#160;protected]</a>
Date: Thu, <span class="m">01</span> Jul <span class="m">2021</span> <span class="m">12</span>:15:50 -0000
Message-ID: &lt;<span class="m">162514175053</span><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="1a342e2a342f2d2a2f222328292d2b2f29222f22292b2b2f5a2f2b2e2a222e2e7f78782b2f">[email&#160;protected]</a>&gt;

Order: 3947963f-1860-44d1-9b9a-4648fed04581 - product: Coffee
Order: ff449e6e-3dfd-48a8-9d5c-79a145d08253 - product: Rice

-------------------------------------------------------------------------------
E-mail Report was sent.
</code></pre></div>

<h3 id="celery-beat">Celery Beat</h3>
<p>We now need to create a periodic task to run this command daily.</p>
<p>Add a new task to <em>core/tasks.py</em>:</p>
<div class="codehilite"><pre><span></span><code><span class="kn">from</span> <span class="nn">celery</span> <span class="kn">import</span> <span class="n">shared_task</span>
<span class="kn">from</span> <span class="nn">celery.utils.log</span> <span class="kn">import</span> <span class="n">get_task_logger</span>
<span class="kn">from</span> <span class="nn">django.core.management</span> <span class="kn">import</span> <span class="n">call_command</span> <span class="c1"># NEW</span>


<span class="n">logger</span> <span class="o">=</span> <span class="n">get_task_logger</span><span class="p">(</span><span class="vm">__name__</span><span class="p">)</span>


<span class="nd">@shared_task</span>
<span class="k">def</span> <span class="nf">sample_task</span><span class="p">():</span>
    <span class="n">logger</span><span class="o">.</span><span class="n">info</span><span class="p">(</span><span class="s2">&quot;The sample task just ran.&quot;</span><span class="p">)</span>


<span class="c1"># NEW</span>
<span class="nd">@shared_task</span>
<span class="k">def</span> <span class="nf">send_email_report</span><span class="p">():</span>
    <span class="n">call_command</span><span class="p">(</span><span class="s2">&quot;email_report&quot;</span><span class="p">,</span> <span class="p">)</span>
</code></pre></div>

<p>So, first we added a <code>call_command</code> import, which is used for programmatically calling django-admin commands. In the new task, we then used the <code>call_command</code> with the name of our custom command as an argument.</p>
<p>To schedule this task, open the <em>core/settings.py</em> file, and update the <code>CELERY_BEAT_SCHEDULE</code> setting to include the new task:</p>
<div class="codehilite"><pre><span></span><code><span class="n">CELERY_BEAT_SCHEDULE</span> <span class="o">=</span> <span class="p">{</span>
    <span class="s2">&quot;sample_task&quot;</span><span class="p">:</span> <span class="p">{</span>
        <span class="s2">&quot;task&quot;</span><span class="p">:</span> <span class="s2">&quot;core.tasks.sample_task&quot;</span><span class="p">,</span>
        <span class="s2">&quot;schedule&quot;</span><span class="p">:</span> <span class="n">crontab</span><span class="p">(</span><span class="n">minute</span><span class="o">=</span><span class="s2">&quot;*/1&quot;</span><span class="p">),</span>
    <span class="p">},</span>
    <span class="s2">&quot;send_email_report&quot;</span><span class="p">:</span> <span class="p">{</span>
        <span class="s2">&quot;task&quot;</span><span class="p">:</span> <span class="s2">&quot;core.tasks.send_email_report&quot;</span><span class="p">,</span>
        <span class="s2">&quot;schedule&quot;</span><span class="p">:</span> <span class="n">crontab</span><span class="p">(</span><span class="n">hour</span><span class="o">=</span><span class="s2">&quot;*/1&quot;</span><span class="p">),</span>
    <span class="p">},</span>
<span class="p">}</span>
</code></pre></div>

<p>Here we added a new entry to the <code>CELERY_BEAT_SCHEDULE</code> called <code>send_email_report</code>. As we did for our previous task, we declared which task it should run -- e.g., <code>core.tasks.send_email_report</code> -- and used a crontab pattern to set the recurrence.</p>
<p>Restart the containers to make sure the new settings become active:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose up -d --build
</code></pre></div>

<p>Open the logs associated with the <code>celery</code> service:</p>
<div class="codehilite"><pre><span></span><code>$ docker-compose logs -f <span class="s1">&#39;celery&#39;</span>
</code></pre></div>

<p>You should see the <code>send_email_report</code> listed:</p>
<div class="codehilite"><pre><span></span><code>celery_1  <span class="p">|</span>  -------------- <span class="o">[</span>queues<span class="o">]</span>
celery_1  <span class="p">|</span>                 .&gt; celery           <span class="nv">exchange</span><span class="o">=</span>celery<span class="o">(</span>direct<span class="o">)</span> <span class="nv">key</span><span class="o">=</span>celery
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span> <span class="o">[</span>tasks<span class="o">]</span>
celery_1  <span class="p">|</span>   . core.tasks.sample_task
celery_1  <span class="p">|</span>   . core.tasks.send_email_report
</code></pre></div>

<p>A minute or so later you should see that the e-mail report is sent:</p>
<div class="codehilite"><pre><span></span><code>celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">12</span>:22:00,071: WARNING/ForkPoolWorker-8<span class="o">]</span> Content-Type: text/plain<span class="p">;</span> <span class="nv">charset</span><span class="o">=</span><span class="s2">&quot;utf-8&quot;</span>
celery_1  <span class="p">|</span> MIME-Version: <span class="m">1</span>.0
celery_1  <span class="p">|</span> Content-Transfer-Encoding: 7bit
celery_1  <span class="p">|</span> Subject: <span class="o">[</span>Django<span class="o">]</span> Order Report <span class="k">for</span> <span class="m">2021</span>-07-01 to <span class="m">2021</span>-07-02
celery_1  <span class="p">|</span> From: <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="65170a0a1125090a0604090d0a1611">[email&#160;protected]</a>
celery_1  <span class="p">|</span> To: <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="0e7a6b7d7a207b7d6b7c4e6b636f6762206d6163">[email&#160;protected]</a>
celery_1  <span class="p">|</span> Date: Thu, <span class="m">01</span> Jul <span class="m">2021</span> <span class="m">12</span>:22:00 -0000
celery_1  <span class="p">|</span> Message-ID: &lt;<span class="m">162514212006</span><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="82acb3b5acb4b2bab2b6b7bbb0bbbbb7b7bab1b7b4bab5b4c2b7b7e0bbbabab1e1b7b6b3b6">[email&#160;protected]</a>&gt;
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span> Order: 3947963f-1860-44d1-9b9a-4648fed04581 - product: Coffee
celery_1  <span class="p">|</span> Order: ff449e6e-3dfd-48a8-9d5c-79a145d08253 - product: Rice
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">12</span>:22:00,071: WARNING/ForkPoolWorker-8<span class="o">]</span> -------------------------------------------------------------------------------
celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">12</span>:22:00,071: WARNING/ForkPoolWorker-8<span class="o">]</span>
celery_1  <span class="p">|</span>
celery_1  <span class="p">|</span> <span class="o">[</span><span class="m">2021</span>-07-01 <span class="m">12</span>:22:00,071: WARNING/ForkPoolWorker-8<span class="o">]</span> E-mail Report was sent.
</code></pre></div>

</main>


</body>
</html>
