# RxAndroidBasic03
RecycleAdapter 를 써서 추가된 데이터만 갱신해준다
'''java
package com.guozhe.android.rxandroidbasic03;

import android.content.Context;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.text.DateFormatSymbols;
import java.util.ArrayList;
import java.util.List;

import io.reactivex.Observable;
import io.reactivex.android.schedulers.AndroidSchedulers;
import io.reactivex.schedulers.Schedulers;


public class MainActivity extends AppCompatActivity {
    private RecyclerView recyclerView;
    RecyclerAdapter adapter;
    Observable<String> observable;
    List<String> data = new ArrayList<>();


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initView();

        adapter = new RecyclerAdapter(this, data);
        recyclerView.setAdapter(adapter);
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        // 캘린더에서 1월~ 12월까지 텍스트를 추출
        DateFormatSymbols dfs = new DateFormatSymbols();
        String[] months = dfs.getMonths();

        observable = Observable.create(emitter -> {
            for(String month : months){
                emitter.onNext(month);
                Thread.sleep(1000);
            }
            emitter.onComplete();
        });
    }

    public void async(View view) {
        observable
                .subscribeOn(Schedulers.io())              // 옵저버블의 thread 를 지정
                .observeOn(AndroidSchedulers.mainThread()) // 옵저버의 thread를 지정
                .subscribe(
                        str -> {
                            data.add(str);
                            // adapter.notifyItemChanged(포지션); // 변경된 데이터만 갱신해준다
                            adapter.notifyItemInserted(data.size()-1); // 추가된 데이터만 갱신해준다
                        },                            // onNext
                        error -> Log.e("Error",error.getMessage()),  // onError
                        () -> {                                          // onComplete
                            data.add("complete!");
                            adapter.notifyItemInserted(data.size()-1);
                        }
                );
    }

    private void initView() {
        recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
    }
}

class RecyclerAdapter extends RecyclerView.Adapter<RecyclerAdapter.Holder> {
    LayoutInflater inflater = null;
    List<String> data = null;

    public RecyclerAdapter(Context context, List<String> data){
        inflater = LayoutInflater.from(context);
        this.data = data;
    }

    @Override
    public Holder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = inflater.inflate(android.R.layout.simple_list_item_1, parent, false);
        return new Holder(view);
    }

    @Override
    public void onBindViewHolder(Holder holder, int position) {
        Log.i("Refresh","~~~~~~~~~~~~~~~~~position="+position);
        holder.textView.setText(data.get(position));
    }

    @Override
    public int getItemCount() {
        return data.size();
    }

    class Holder extends RecyclerView.ViewHolder {
        TextView textView;
        public Holder(View itemView) {
            super(itemView);
            textView = (TextView) itemView.findViewById(android.R.id.text1);
        }
    }
}
