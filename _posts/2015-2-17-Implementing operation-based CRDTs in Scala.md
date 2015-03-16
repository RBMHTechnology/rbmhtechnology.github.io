---
layout: post
title: Implementing operation-based CRDTs in Scala
author: mkrasser
tag: eventuate
---

In a <a href="http://rbmhtechnology.github.io/Event%20Sourcing%20at%20Global%20Scale/">previous post</a> I described how actor state can be globally replicated via event sourcing. Keeping replicas available for writes during a network partition requires resolution of conflicting writes when the partition heals. In this context, <a href="http://en.wikipedia.org/wiki/Conflict-free_replicated_data_type">conflict-free replicated data types</a> (CRDTs) have already been mentioned.

The <a href="https://github.com/RBMHTechnology/eventuate">Eventuate</a> project now provides implementations of <a href="http://rbmhtechnology.github.io/eventuate/user-guide.html#commutative-replicated-data-types">operation-based CRDTs</a> (CmRDTs) as specified in the paper <a href="http://hal.upmc.fr/docs/00/55/55/88/PDF/techreport.pdf">A comprehensive study of Convergent and Commutative Replicated Data Types</a> by Marc Shapiro et. al. Currently, the following CmRDTs are implemented (more coming soon):

<ul>
  <li><a href="https://github.com/RBMHTechnology/eventuate/blob/blog-crdt-code/src/main/scala/com/rbmhtechnology/eventuate/crdt/Counter.scala">Counter</a> (specification 5)</li>
  <li><a href="https://github.com/RBMHTechnology/eventuate/blob/blog-crdt-code/src/main/scala/com/rbmhtechnology/eventuate/crdt/MVRegister.scala">MV-Register</a> (op-based version of specification 10) </li>
  <li><a href="https://github.com/RBMHTechnology/eventuate/blob/blog-crdt-code/src/main/scala/com/rbmhtechnology/eventuate/crdt/ORSet.scala">OR-Set</a> (specification 15)</li>
</ul>

Basis for the implementation is a <a href="http://rbmhtechnology.github.io/eventuate/architecture.html#event-logs">replicated event log</a> and <a href="http://rbmhtechnology.github.io/eventuate/architecture.html#event-sourced-actors">event-sourced actors</a>. A replicated event log

<ul>
  <li>supports the reliable broadcast of update-operations.</li>
  <li>chooses AP from <a href="http://en.wikipedia.org/wiki/CAP_theorem">CAP</a> i.e. applications can continue writing to a local replica during a network partition.</li>
  <li>preserves causal ordering of events which satisfies all <em>downstream</em> preconditions of the CmRDTs specified in the paper.</li>
</ul>

Event-Sourced Actors

<ul>
  <li>generate vector timestamps that are used by some CmRDTs to determine whether any two updates are concurrent or causally related.</li>
  <li>maintain CmRDT instances in-memory. Their state can be recovered by replaying update-operations (optionally starting from a snapshot).</li>
</ul>

CmRDT update specifications in the paper have a close relationship to the command and event handler of an event-sourced actor. A CmRDT update has two phases. The first phase is called <em>atSource</em>: 

<blockquote>
	It takes its arguments from the operation invocation; it is not allowed to make side effects; it may compute results, returned to the caller, and/or prepare arguments for the second phase.
</blockquote>

To <em>prepare arguments for the second phase</em>, update-operation events are generated and persisted by the command handler. The second update phase is called <em>downstream</em>. It executes

<blockquote>
  … immediately at the source, and asynchronously, at all other replicas; it can not return results.
</blockquote>

The <em>downstream</em> phase executes by consuming update-operation events in the event handler. Not only does the local replica consume the events it generated and persisted but also all other replicas on the same replicated event log consume these events (= reliable broadcast). Consumed update-operation events finally change the state of CmRDTs that are maintained by event-sourced actors in-memory.

<h2>USAGE</h2>

Applications that want to use these CmRDTs don’t need to interact directly with event-sourced actors. Instead, Eventuate provides service interfaces for reading and updating CmRDTs. There’s a service interface for each supported CmRDT type.

<h3>COUNTER</h3>

<code>Counter[A]</code> CmRDTs are managed by <code>CounterService[A]</code> which provides the following read and update operations.

<div class="highlight"><pre><code class="language-scala" data-lang="scala"><span class="k">class</span> <span class="nc">CounterService</span><span class="o">[</span><span class="kt">A</span> <span class="kt">:</span> <span class="kt">Integral</span><span class="o">](</span><span class="k">val</span> <span class="n">replicaId</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="k">val</span> <span class="n">log</span><span class="k">:</span> <span class="kt">ActorRef</span><span class="o">)</span> <span class="o">{</span>
  <span class="k">def</span> <span class="n">value</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">A</span><span class="o">]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
  <span class="k">def</span> <span class="n">update</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="n">delta</span><span class="k">:</span> <span class="kt">A</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">A</span><span class="o">]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
<span class="o">}</span></code></pre></div>

The <code>value</code> method reads a counter value, <code>update</code> updates a counter value with a given <code>delta</code>. Counter instances are identified by application-defined <code>id</code>s. Each <code>CounterService</code> replica must have a unique <code>replicaId</code> and a reference to the replicated event <code>log</code>. The following example creates and uses a <code>CounterService[Int]</code> for reading and updating <code>Counter[Int]</code> CmRDTs.

<div class="highlight"><pre><code class="language-scala" data-lang="scala"><span class="k">import</span> <span class="nn">akka.actor.ActorRef</span>
<span class="k">import</span> <span class="nn">com.rbmhtechnology.eventuate.crdt.CounterService</span>

<span class="k">val</span> <span class="n">eventLog</span><span class="k">:</span> <span class="kt">ActorRef</span> <span class="o">=</span> <span class="o">...</span>
<span class="k">val</span> <span class="n">counterService</span> <span class="k">=</span> <span class="k">new</span> <span class="nc">CounterService</span><span class="o">[</span><span class="kt">Int</span><span class="o">](</span><span class="s">&quot;counter-replica-1&quot;</span><span class="o">,</span> <span class="n">eventLog</span><span class="o">)</span>

<span class="c1">// counter-1 usage</span>
<span class="n">counterService</span><span class="o">.</span><span class="n">update</span><span class="o">(</span><span class="s">&quot;counter-1&quot;</span><span class="o">,</span> <span class="mi">11</span><span class="o">)</span> <span class="c1">// increment</span>
<span class="n">counterService</span><span class="o">.</span><span class="n">update</span><span class="o">(</span><span class="s">&quot;counter-1&quot;</span><span class="o">,</span> <span class="o">-</span><span class="mi">2</span><span class="o">)</span> <span class="c1">// decrement</span>
<span class="n">counterService</span><span class="o">.</span><span class="n">value</span><span class="o">(</span><span class="s">&quot;counter-1&quot;</span><span class="o">)</span>      <span class="c1">// read</span>

<span class="c1">// counter-2 usage</span>
<span class="n">counterService</span><span class="o">.</span><span class="n">value</span><span class="o">(</span><span class="s">&quot;counter-2&quot;</span><span class="o">)</span>     <span class="c1">// read</span>
<span class="n">counterService</span><span class="o">.</span><span class="n">update</span><span class="o">(</span><span class="s">&quot;counter-2&quot;</span><span class="o">,</span> <span class="mi">3</span><span class="o">)</span> <span class="c1">// increment</span></code></pre></div>

Counters are created on demand if referenced the first time by an update operation.

<h3>MV-REGISTER</h3>

<code>MVRegister[A]</code> CmRDTs are managed by <code>MVRegisterService[A]</code> which provides the following read and update operations.

<div class="highlight"><pre><code class="language-scala" data-lang="scala"><span class="k">class</span> <span class="nc">MVRegisterService</span><span class="o">[</span><span class="kt">A</span><span class="o">](</span><span class="k">val</span> <span class="n">replicaId</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="k">val</span> <span class="n">log</span><span class="k">:</span> <span class="kt">ActorRef</span><span class="o">)</span> <span class="o">{</span>
  <span class="k">def</span> <span class="n">value</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">Set</span><span class="o">[</span><span class="kt">A</span><span class="o">]]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
  <span class="k">def</span> <span class="n">set</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="n">value</span><span class="k">:</span> <span class="kt">A</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">Set</span><span class="o">[</span><span class="kt">A</span><span class="o">]]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
<span class="o">}</span></code></pre></div>

<h3>OR-SET</h3>

<code>ORSet[A]</code> CmRDTs are managed by <code>ORSetService[A]</code> which provides the following read and update operations.

<div class="highlight"><pre><code class="language-scala" data-lang="scala"><span class="k">class</span> <span class="nc">ORSetService</span><span class="o">[</span><span class="kt">A</span><span class="o">](</span><span class="k">val</span> <span class="n">replicaId</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="k">val</span> <span class="n">log</span><span class="k">:</span> <span class="kt">ActorRef</span><span class="o">)</span> <span class="o">{</span>
  <span class="k">def</span> <span class="n">value</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">Set</span><span class="o">[</span><span class="kt">A</span><span class="o">]]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
  <span class="k">def</span> <span class="n">add</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="n">entry</span><span class="k">:</span> <span class="kt">A</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">Set</span><span class="o">[</span><span class="kt">A</span><span class="o">]]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
  <span class="k">def</span> <span class="n">remove</span><span class="o">(</span><span class="n">id</span><span class="k">:</span> <span class="kt">String</span><span class="o">,</span> <span class="n">entry</span><span class="k">:</span> <span class="kt">A</span><span class="o">)</span><span class="k">:</span> <span class="kt">Future</span><span class="o">[</span><span class="kt">Set</span><span class="o">[</span><span class="kt">A</span><span class="o">]]</span> <span class="k">=</span> <span class="o">{</span> <span class="o">...</span> <span class="o">}</span>
<span class="o">}</span></code></pre></div>

<h2>RUNNING</h2>

A running OR-Set example, based on a multi-JVM test, can be found <a href="https://github.com/RBMHTechnology/eventuate/blob/blog-crdt-code/src/multi-jvm/scala/com/rbmhtechnology/eventuate/crdt/ReplicatedORSetSpec.scala">here</a>.

