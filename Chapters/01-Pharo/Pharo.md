## Getting Started in Pharo
Example_ which is a free and online book available from [htp://books.pharo.org](htp://books.pharo.org)  and the Pharo mooc available at [http://mooc.pharo.org](http://mooc.pharo.org).
WAKom stop.
WAKom startOn: 8081.
    instanceVariableNames: 'count'
    classVariableNames: ''
    package: 'WebCounter'
   super initialize.
   count := 0
    count := count + 1
    count := count - 1
    html heading: count
    WAAdmin register: self asApplicationAt: 'webcounter'
    html heading: count.
    html anchor
        callback: [ self increase ];
        with: '++'.
    html space.
    html anchor
        callback: [ self decrease ];
        with: '--'