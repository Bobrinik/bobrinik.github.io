---
layout: post
title: "Modulo Hashing Visualizer"
date: 2025-11-29
tags: [Deep Dive, Tutorial, System Design]
---
## Problem
The current database cannot handle the volume of incoming write requests. During peak times, there are too many write/update requests incoming per second, so requests are taking longer to execute. You can buffer requests, but if the database cannot fulfill them faster than they arrive, it will overflow. Let’s say you work in a bank and cannot afford to drop any requests.



## Solution
To scale writes, you can either use a bigger database (scale vertically) or use multiple databases (scale horizontally). Let's say you want to scale horizontally. So you add an extra database. Now, you need to figure out how to forward requests to multiple DBs. 

You can do it in a round-robin style. The issue is that requests for user X are persisted across different DBs, which makes querying all records for X slower (we need to query all  DBs to get results) and makes enforcing table constraints more difficult (cross-database referential integrity is handled outside the database engine). Since round-robin is not working for us because we lose referential integrity this way, we need to route requests so that user X always goes to Database 1, so that all of user X's data is located on Database 1, and the database engine can perform referential-integrity checks for that user.

One way to achieve this is to use the modulo operator. We can take the modulo of `user_id` and use the result to determine which database to map our user to. Here's an example of how it can be done. We can take our ID, convert it to an integer, and perform a modulo operation on it.

![](/assets/images/2025-11-29-modulo-hashing/image.png)

It works nicely as long as your ids are evenly distributed. If our ids are evenly distributed, then each database receives the same number of users. Let's check if our UUIDs are evenly distributed.

![](/assets/images/2025-11-29-modulo-hashing/uuid_even_distribution.png)

Pretty much evenly distributed, there seems to be some noise around the 2nd bucket, but it will smooth out as numbers increase.


## Now what's the problem with modulo hashing?
Problems with this approach start when we want to re-scale our database. Because when we rescale our database, instead of doing `modulo 5` we do `modulo 6` and records that were mapped to Database 1 are now going to be mapped to Database 6 and it will need to happen for many records.

```
5 % 5 -> 0
6 % 5 -> 1
7 % 5 -> 2

5 % 6 -> 1
6 % 6 -> 0
7 % 6 -> 1
```


<div id="modulo-hashing-root"></div>

<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script src="https://cdn.tailwindcss.com"></script>

<script type="text/babel">
const { useState, useMemo } = React;

// Simple icon components
const Database = ({ className }) => (
  <svg className={className} fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <ellipse cx="12" cy="5" rx="9" ry="3" />
    <path d="M21 12c0 1.66-4 3-9 3s-9-1.34-9-3" />
    <path d="M3 5v14c0 1.66 4 3 9 3s9-1.34 9-3V5" />
  </svg>
);

const ArrowRight = ({ className }) => (
  <svg className={className} fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M5 12h14M12 5l7 7-7 7" />
  </svg>
);

const AlertTriangle = ({ className }) => (
  <svg className={className} fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
  </svg>
);

const RefreshCcw = ({ className }) => (
  <svg className={className} fill="none" viewBox="0 0 24 24" stroke="currentColor" strokeWidth={2}>
    <path strokeLinecap="round" strokeLinejoin="round" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
  </svg>
);

const ModuloHashingVisualizer = () => {
  const [numKeys, setNumKeys] = useState(20);
  const [shardsBefore, setShardsBefore] = useState(4);
  const [shardsAfter, setShardsAfter] = useState(5);

  const getShardColor = (index) => {
    const colors = [
      'bg-blue-900/50 border-blue-500 text-blue-300',
      'bg-green-900/50 border-green-500 text-green-300',
      'bg-purple-900/50 border-purple-500 text-purple-300',
      'bg-orange-900/50 border-orange-500 text-orange-300',
      'bg-pink-900/50 border-pink-500 text-pink-300',
      'bg-teal-900/50 border-teal-500 text-teal-300',
      'bg-yellow-900/50 border-yellow-500 text-yellow-300',
      'bg-indigo-900/50 border-indigo-500 text-indigo-300',
      'bg-red-900/50 border-red-500 text-red-300',
      'bg-gray-700/50 border-gray-500 text-gray-300',
    ];
    return colors[index % colors.length];
  };

  const data = useMemo(() => {
    let movedCount = 0;
    const records = [];

    for (let i = 0; i < numKeys; i++) {
      const prevShard = i % shardsBefore;
      const newShard = i % shardsAfter;
      const hasMoved = prevShard !== newShard;
      
      if (hasMoved) movedCount++;

      records.push({
        id: i,
        prevShard,
        newShard,
        hasMoved
      });
    }

    return { records, movedCount };
  }, [numKeys, shardsBefore, shardsAfter]);

  const percentMoved = ((data.movedCount / numKeys) * 100).toFixed(1);

  return (
    <div className="p-6 max-w-4xl mx-auto bg-[#191919] rounded-xl border border-[#333] font-sans">
      <div className="mb-6">
        <h2 className="text-2xl font-bold text-[#e0e0e0] flex items-center gap-2">
          <Database className="w-6 h-6 text-[#7cb3f3]" />
          The Modulo Hashing Problem
        </h2>
        <p className="text-[#9a9a9a] mt-2">
          Visualize why simple <code className="bg-[#252525] px-1 rounded text-[#e06c75]">hash(key) % N</code> fails when scaling.
          When the number of shards (N) changes, the result of the modulo operation changes for most keys.
        </p>
      </div>

      {/* Controls */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6 bg-[#252525] p-4 rounded-lg border border-[#333] mb-6">
        <div>
          <label className="block text-sm font-semibold text-[#e0e0e0] mb-2">Total Records (Keys)</label>
          <input 
            type="range" min="10" max="100" 
            value={numKeys} 
            onChange={(e) => setNumKeys(parseInt(e.target.value))}
            className="w-full accent-[#7cb3f3]"
          />
          <div className="text-right text-[#9a9a9a] font-mono">{numKeys} Keys</div>
        </div>

        <div>
          <label className="block text-sm font-semibold text-[#e0e0e0] mb-2">Current Shard Count (N)</label>
          <div className="flex items-center gap-2">
            <button 
              onClick={() => setShardsBefore(Math.max(1, shardsBefore - 1))}
              className="px-3 py-1 bg-[#2f2f2f] hover:bg-[#3a3a3a] text-[#e0e0e0] rounded border border-[#333]"
            >-</button>
            <span className="font-mono text-lg w-8 text-center text-[#e0e0e0]">{shardsBefore}</span>
            <button 
              onClick={() => setShardsBefore(shardsBefore + 1)}
              className="px-3 py-1 bg-[#2f2f2f] hover:bg-[#3a3a3a] text-[#e0e0e0] rounded border border-[#333]"
            >+</button>
          </div>
        </div>

        <div>
          <label className="block text-sm font-semibold text-[#e0e0e0] mb-2">New Shard Count (N+1)</label>
          <div className="flex items-center gap-2">
            <button 
              onClick={() => setShardsAfter(Math.max(1, shardsAfter - 1))}
              className="px-3 py-1 bg-[#2f2f2f] hover:bg-[#3a3a3a] text-[#e0e0e0] rounded border border-[#333]"
            >-</button>
            <span className="font-mono text-lg w-8 text-center text-[#e0e0e0]">{shardsAfter}</span>
            <button 
              onClick={() => setShardsAfter(shardsAfter + 1)}
              className="px-3 py-1 bg-[#2f2f2f] hover:bg-[#3a3a3a] text-[#e0e0e0] rounded border border-[#333]"
            >+</button>
          </div>
        </div>
      </div>

      {/* Impact Stats */}
      <div className={`p-4 rounded-lg border mb-6 flex items-center justify-between transition-colors duration-300 ${parseInt(percentMoved) > 30 ? 'bg-red-900/20 border-red-800' : 'bg-green-900/20 border-green-800'}`}>
        <div className="flex items-center gap-3">
          {parseInt(percentMoved) > 30 ? <AlertTriangle className="w-6 h-6 text-red-400" /> : <RefreshCcw className="w-6 h-6 text-green-400" />}
          <div>
            <div className="font-bold text-[#e0e0e0]">Reshuffle Impact</div>
            <div className="text-sm text-[#9a9a9a]">Keys that must be moved to a new server</div>
          </div>
        </div>
        <div className="text-right">
          <div className={`text-3xl font-bold ${parseInt(percentMoved) > 30 ? 'text-red-400' : 'text-green-400'}`}>
            {percentMoved}%
          </div>
          <div className="text-sm text-[#9a9a9a]">{data.movedCount} of {numKeys} records</div>
        </div>
      </div>

      {/* Visualization Grid */}
      <div className="bg-[#252525] p-4 rounded-lg border border-[#333]">
        <h3 className="text-sm font-semibold text-[#9a9a9a] uppercase tracking-wide mb-4">Record Mapping Visualization</h3>
        
        <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-3">
          {data.records.map((record) => (
            <div 
              key={record.id}
              className={`relative p-3 rounded-lg border-2 transition-all duration-500 ${
                record.hasMoved 
                  ? 'border-red-500 bg-red-900/30 opacity-100' 
                  : 'border-[#333] bg-[#1e1e1e] opacity-60'
              }`}
            >
              <div className="flex justify-between items-center mb-2">
                <span className="text-xs font-bold text-[#6b6b6b]">KEY {record.id}</span>
                {record.hasMoved && (
                  <span className="text-[10px] font-bold bg-red-900/50 text-red-400 px-1.5 py-0.5 rounded">MOVED</span>
                )}
              </div>

              <div className="flex items-center gap-2">
                <div className={`flex-1 text-center py-1 rounded text-xs font-mono border ${getShardColor(record.prevShard)}`}>
                  S{record.prevShard}
                </div>
                <ArrowRight className={`w-4 h-4 ${record.hasMoved ? 'text-red-400' : 'text-[#6b6b6b]'}`} />
                <div className={`flex-1 text-center py-1 rounded text-xs font-mono border ${getShardColor(record.newShard)}`}>
                  S{record.newShard}
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>


    </div>
  );
};

const root = ReactDOM.createRoot(document.getElementById('modulo-hashing-root'));
root.render(<ModuloHashingVisualizer />);
</script>

## Conclusion
Modulo hashing is simple and works well for static systems, but it becomes problematic when you need to scale. For some cases the number of records that need moving can go up as high as 93%. There are different ways of solving it. One way is to use consisten-hashing or lookup table. 