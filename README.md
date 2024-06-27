<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>

<h1>Class with threading problem and concurency test</h1>

<p>This repository demonstrates a common issue in concurrent programming: race conditions. It includes a simple class with a threading problem and a corresponding test to expose the issue. Additionally, it provides a fix for the race condition.</p>

<h2>ClassWithThreadingProblem</h2>

<p>The <code>ClassWithThreadingProblem</code> class increments an <code>int</code> variable without any synchronization, leading to potential race conditions when accessed by multiple threads concurrently.</p>

<h3>Problematic Implementation</h3>

<pre><code>public class ClassWithThreadingProblem
{
    private int nextId = 0;

    public int TakeNextId()
    {
        return nextId++;
    }

    public int LastId
    {
        get { return nextId; }
    }
}
</code></pre>

<h3>Test to Expose the Issue</h3>

<p>The <code>ClassWithThreadingProblemTest</code> class runs a test to demonstrate the race condition. It creates two threads that call the <code>TakeNextId</code> method simultaneously, which can lead to incorrect incrementing of the <code>nextId</code> variable.</p>

<h4>ClassWithThreadingProblemTest.cs</h4>

<pre><code>using System;
using System.Threading;
using NUnit.Framework;

namespace Example
{
    [TestFixture]
    public class ClassWithThreadingProblemTest
    {
        [Test]
        public void TwoThreadsShouldFailEventually()
        {
            var classWithThreadingProblem = new ClassWithThreadingProblem();
            var runnable = new ThreadStart(() => classWithThreadingProblem.TakeNextId());

            for (int i = 0; i < 50000; ++i)
            {
                int startingId = classWithThreadingProblem.LastId;
                int expectedResult = 2 + startingId;

                Thread t1 = new Thread(runnable);
                Thread t2 = new Thread(runnable);
                t1.Start();
                t2.Start();
                t1.Join();
                t2.Join();

                int endingId = classWithThreadingProblem.LastId;

                if (endingId != expectedResult)
                    return;
            }

            Assert.Fail("Should have exposed a threading issue but it did not.");
        }
    }
}
</code></pre>

<h3>Fixed Implementation</h3>

<p>The race condition can be fixed by synchronizing access to the <code>TakeNextId</code> method using a lock.</p>

<pre><code>public class ClassWithThreadingProblem
{
    private int nextId = 0;
    private readonly object lockObject = new object();

    public int TakeNextId()
    {
        lock (lockObject)
        {
            return nextId++;
        }
    }

    public int LastId
    {
        get { return nextId; }
    }
}
</code></pre>

</body>
</html>
