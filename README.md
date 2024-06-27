# Performance and Concurency Testing


The C# code in ClassWithThreadingProblem is incrementing the nextId variable without any synchronization. This means that if multiple threads access takeNextId() concurrently, they might both read the same value of nextId before either increments it, causing a race condition. This results in nextId being incremented only once instead of twice.

public class ClassWithThreadingProblem
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

using System;
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

Fix for that race condition 

public class ClassWithThreadingProblem
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


