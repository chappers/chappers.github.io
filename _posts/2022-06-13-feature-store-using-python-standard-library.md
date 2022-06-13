---
layout: post
category:
tags:
tagline:
---

As a followup to writing lightweight feature store, I decided to test out "what-if" we were to write one in Python using only standard libraries - how difficult was it, if we didn't care about performance too much?

## Requirements

* Support time travel in offline feature retrieval
* Support extracting data from snapshot
* Support online feature retrieval

It turns out this is all achievable in under 200 lines of code!

Hopefully the code explains itself. At some stage I'll post it on github.

## Design notes

* Used jsonlines as the data exchange format, because `json` is in the Python standard libraries
* Used `dbm` as a key-value store, where the online features are served as `json` objects

We make assumptions about the folder structure, that it is partitioned by date, and then by time so that each folder only has each entity "once".

## Code

```py
import dbm
import json
import tempfile
from dataclasses import dataclass
from datetime import datetime, timedelta
from io import StringIO
from pathlib import Path
from typing import Any, Dict, List, Optional, Union

try:
    import pandas
except ImportError:
    pandas = None

@dataclass
class OnlineFeatureRequest:
    entity_name:str
    entity_value: Any
    feature_set: str

@dataclass
class HistoricalFeatureRequest:
    entity_name: str
    entity_value: Any
    feature_set: str
    event_timestamp: Optional[datetime] = None
    ttl: Optional[timedelta] = None
    extra: Optional[dict] = None

@dataclass
class HistoricalMultiEntityFeatureRequest:
    entities: Dict[str, Any]

    feature_set: str
    event_timestamp: Optional[datetime] = None
    ttl: Optional[timedelta] = None
    extra: Optional[dict] = None

@dataclass
class SnapshotFeatureSetRequest:
    entity_name: str
    feature_set: str
    snapshot_timestamp: datetime
    ttl: Optional[timedelta] = None

class FeatureStore(object):
    CACHE_NAME = "cache"

    def __init__(
        self,
        store_path="data/"
    ):
        self.store_path = Path(store_path)

    def _custom_file_filter_sort(
        self,
        list_files: List[str],
        event_timestamp: Optional[datetime] = None,
        ttl: Optional[timedelta] = None,
        event_timestamp_index=-2
    ):
        if event_timestamp_index not in [-3, -2]:
            raise ValueError(f"Event timestamp index should only be -2, or -3, got: {event_timestamp_index}.")
        
        if ttl and not event_timestamp:
            raise ValueError("If ttl is provided then event timestamp cannot be None.")
        
        def custom_sort_key(x):
            key_x = str(x).rsplit("/", abs(event_timestamp_index))
            return [key_x[idx] for idx in range(event_timestamp_index, 0)]
        
        def custom_filter_function(x, event_timestamp=event_timestamp, ttl=ttl):
            if event_timestamp:
                key = str(x).rsplit("/", abs(event_timestamp_index))
                record_timestamp = datetime.strptime(key_x[event_timestamp_index], "%Y-%m-%d").replace(tzinfo=None)
                event_timestamp = event_timestamp.replace(tzinfo=None)

                if ttl:
                    return record_timestamp <= event_timestamp and record timestamp >= (event_timestamp - ttl)
                else:
                    return record_timestamp <= event_timestamp
            else:
                return True

        if event_timestamp:
            return sorted(filter(custom_filter_function, list_files), key=custom_sort_key, reverse=True)
        else:
            return sorted(list_files, key=custom_sort_key, reverse=True)
    
    def _build_jsonl_path(
        self, feature_set: str, event_timestamp: Optional[datetime] = None, ttl: Optional[timedelta] = None
    ):
        feature_set_path = self.store_path.joinpath(feature_set)

        # make sure we capture the different ways jsonl files can be stored
        return self._custom_file_filter_sort(
            feature_set_path.glob("*/*.jsonl"), event_timestamp, ttl, -2
        ) + self._custom_file_filter_sort(feature_set_path.glob("*/*.jsonl"), event_timestamp, ttl, -3)
    
    def get_historical_feature(self, feature_request: HistoricalFeatureRequest):
        list_jsonl = self._build_jsonl_path(
            feature_request.feature_set, feature_request.event_timestamp, feature_request.ttl
        )

        # find the precise entry - it is expensive to loop through, but this way works...
        for jsonl_path in list_jsonl:
            for record in open(jsonl_path, "r").readlines():
                record_dict = json.loads(record)
                if record_dict[feature_request.entity_name] == feature_request.entity_value:
                    return json.dumps({**record_dict, **featuee_request.extra})

    def get_historical_multi_entity_feature(self, feature_request: HistoricalFeatureRequest):
        list_jsonl = self._build_jsonl_path(
            feature_request.feature_set, feature_request.event_timestamp, feature_request.ttl
        )

        # find the precise entry - it is expensive to loop through, but this way works...
        for jsonl_path in list_jsonl:
            for record in open(jsonl_path, "r").readlines():
                record_dict = json.loads(record)
                check_all_entities = all(
                    [
                        (entity_key, entity_value) in record_dict.items()
                        for entity_key, entity_value in feature_request.entities.items()
                    ]
                )
                if check_all_entities:
                    return json.dumps({**record_dict, **featuee_request.extra})

    def get_online_feature(self, feature_request: OnlineFeatureRequest, cache_location: str):
        cache_anme = str(Path(cache_location).joinpath(feature_request.feature_set).joinpath(self.CACHE_NAME))
        with dbm.open(cache_name, "r") as db:
            return json.loads(db[feature_request.entity_value])

    def export_historical_features(
        self,
        batch_request: List[Union[HistoricalFeatureRequest, HistoricalMultiEntityFeatureRequest]],
        path: Optional[str] = None
    ):
        """
        takes batch request object
        """
        if path:
            filepath = Path(path)
            filepath.parent.mkdir(parent=True, exist_ok=True)
            output_file = open(path, "w")
        else:
            output_file = StringIO()

        for feature_request in batch_request:
            if type(feature_request) is HistoricalFeatureRequest:
                historical_output = self.get_historical_feature(feature_request)
            if type(feature_request) is HistoricalMultiEntityFeatureRequest:
                historical_output = self.get_historical_multi_entity_feature(feature_request)
            if historical_output:
                output_file.write(f"{historical_output}\n")
        
        if not path and pandas:
            output_file.seek(0)
            return pandas.read_json(output_file, lines=True)

    def export_snapshot_feature(
        self, snapshot_request: SnapshotFeatureSetRequest, path: Optional[str]=None, cache_location=None
    ):
        """
        Exports things to an online store based on configuration - we avoid OOM but spooling onto disc using DBM
        Can be optimized...probably
        """
        refersh_location = temptfile.TemporaryDirectory()
        cleanup = cache_location is None

        if path:
            filepath = Path(path)
            filepath.parent.mkdir(parent=True, exist_ok=True)
            output_file = open(path, "w")
        else:
            output_file = StringIO()

        list_jsonl = list(
            self._build_jsonl_path(
                snapshot_request.feature_set, snapshot_request.snapshot_timestamp, snapshot_request.ttl
            )
        )

        if not cache_location:
            cache_location = tempfile.TemporaryDirectory()
        cache_name = Path(cache_location).joinpath(snapshot_request.feature_set).joinpath(self.CACHE_NAME)
        refresh_name = Path(refresh_location.name).joinpath(snapshot_request.feature_set).joinpath(self.CACHE_NAME)
        cache_name.parent.mkdir(parents=True, exist_ok=True)
        refresh_name.parent.mkdir(parents=True, exist_ok=True)
        with dbm.open(str(cache_name), "c") as db:
            with dbm.open(str(refresh_name), "c") as refresh_db:
                for jsonl_path in list_jsonl:
                    for record in open(jsonl_path, "r").readlines():
                        entity_value = json.loads(record)[snapshot_request.entity_name]
                        if refresh_db.get(str(entity_value)):
                            continue
                        output_file.write(f"{record}\n")
                        db[str(entity_value)] = self.CACHE_NAME if cleanup else record

        if cleanup:
            cache_location.cleanup()
        refresh_location.cleanup()

        if not path and pandas:
            output_file.seek(0)
            return pandas.read_json(output_file, lines=True)
```